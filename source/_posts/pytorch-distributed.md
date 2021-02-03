---
title: pytorch分布式训练初探
date: 2019-10-11 16:52:27
tags:
- distributed training
categories: 
- pytorch


---

## 基础概念：

### 进程、线程

进程即process，线程即thread，从知乎的回答上看，可以了解到，线程是比进程更细粒度的划分，二者均表示占用资源（CPU、内存、IO等）的时间



### world_size, rank

world_size本质就是指 **进程总数**，而rank则指的是 **当前的进程**

- 如果是单机多卡的模式，则world_size可以认为是 该卡的GPU总数，rank可以认为是每个GPU对应的进程（如果一个GPU开一个进程的话）
- 如果是多机多卡的模式，则world_size可以认为是 **所有卡加起来的GPU总数**，rank可以认为是每个GPU对应的进程（如果一个GPU开一个进程的话）。rank必须是exclusive的，否则会有重叠。**但是此时要注意，如果需要手动在不同的node上面分别开启terminal来实现多进程，那么在指定cuda device(`torch.cuda.set_device`)时，必须用那个node上面的相对rank，因为每个node 上面的gpu都是从0开始计数的**。
- 如果要一个rank（一个进程）同时使用好几个GPU的话，那么总的world_size则为total_num_gpus / gpus_per_rank，此时由于是 **几个GPU共用一个进程，可能会导致读取速度下降**。 **建议还是一个GPU一个进程**

## launch 与 spawn的区别

- `torch.distributed.launch`本质上通过`torch/distributed/launch.py`文件来 **先生成对应个数的subprocess**，再分别在对应的subprocess中运行

- 而`torch.multiprocessing.spawn`则是通过调用python的`multiprocessing`的包来生成 **Process**，然后分别在Process中运行对应的函数。参数如下：

  - fn (function) – Function is called as the entrypoint of the spawned process. This function must be defined at the top level of a module so it can be pickled and spawned. This is a requirement imposed by multiprocessing.

    The function is called as fn(i, *args), where i is the process index and args is the passed through tuple of arguments.

  - args (tuple) – Arguments passed to fn.

  - nprocs (int) – Number of processes to spawn.

  - join (bool) – Perform a blocking join on all processes.

  - daemon (bool) – The spawned processes’ daemon flag. If set to True, daemonic processes will be created.

  **因此更适合用来在一个node上运行`nprocs`个进程，分别对应那个node上的所有GPU，一个GPU一个Process**

  > The torch.multiprocessing package also provides a `spawn` function in [`torch.multiprocessing.spawn()`](https://pytorch.org/docs/stable/multiprocessing.html#torch.multiprocessing.spawn). This helper function can be used to spawn multiple processes. It works by passing in the function that you want to run and spawns N processes to run it. This can be used for multiprocess distributed training as well.



- 上述两者本质都是相同的（都是通过产生新的进程，然后把任务放到其中运行），**因此都需要在对应的子任务（launch对应的是那个待运行的.py文件，spawn这个对应的是参数中的`fn`）中设定`init_process_group`初始化distributed running**







## 单机多卡的方式

- 因为服务器用的是slurm，总共只有3个node，因此用满6个gpu大概率也只是单机上的（虽然通常不会要求用满6个，申请3-4个gpu的时候大概率是在一个node上的），用 **单机多卡的方式**训练会比较好
- **默认会把rank 0作为master node**

### 法1：手动开进程

即在一个shell文件中放入多条`srun`指令，来实现每一条`srun`指令对应一个GPU。这样最稳（因为可以通过添加`--exclusive`来实现互不占用，`CUDA_VISIBLE_DEVICES`也只会显示对应的GPU



### 法2：用`torch.distributed.launch`辅助开启进程

- 需要注意，每个subprocess中的`CUDA_VISIBLE_DEVICES`都是能够看到 **整个任务所分配的所有GPU**

- 单机多卡对应的设置如下即可（MaterPort等默认是用localhost，MasterPort随机生成一个？）

  ```bash
  python -m torch.distributed.launch --use_env xxx.py
  ```

- 多机多卡对应的设置参考如下，来自[torch.distributed.launch]( https://github.com/pytorch/pytorch/blob/master/torch/distributed/launch.py )

  ```bash
  
  Node 1: *(IP: 192.168.1.1, and has a free port: 1234)*
  ::
      >>> python -m torch.distributed.launch --nproc_per_node=NUM_GPUS_YOU_HAVE
                 --nnodes=2 --node_rank=0 --master_addr="192.168.1.1"
                 --master_port=1234 YOUR_TRAINING_SCRIPT.py (--arg1 --arg2 --arg3
                 and all other arguments of your training script)
  Node 2:
  ::
      >>> python -m torch.distributed.launch --nproc_per_node=NUM_GPUS_YOU_HAVE
                 --nnodes=2 --node_rank=1 --master_addr="192.168.1.1"
                 --master_port=1234 YOUR_TRAINING_SCRIPT.py (--arg1 --arg2 --arg3
                 and all other arguments of your training script)
  ```

  



### 法3：用`torch.multiprocessing.spawn`来辅助开启进程

- 在`fn`参数对应的function中，每个function都 **只能看到对应rank（即GPU id），因此也比较安全**





## `torch.utils.data.distributed.DistributedSampler` 理解

- 特点概括：

  1. 能够让各个rank（各个进程）都拿到不一样的subset，从而能够保证一个epoch下来，所有Process拿到的数据都是unique的。

  2. 为了保证每个epoch**都能把所有数据遍历一次**，一个epoch中所有进程加起来的input **可能会超过**原来Dataset的size，原则上多出来的部分从 **原Dataset中index小的部分增加起来**。

  3. 可不设定world size（总进程数），rank（当前进程），这两个都可以直接用`torch.distributed`包的函数获得。最后`__iter__`中的indices是根据这两个确定的。（也是保证每个process对应的indices都不一样）

  4. 若要达到shuffle的效果，则需要在每个epoch的循环中使用下列命令：

     ```python
     train_sampler.set_epoch(epoch)
     ```

     

- sampler只是sampler，而非batch_sampler（即 **只提供一个list of all indices，而非list of list of indices per batch）**，因此在后续的`Dataloader`中应该放到`sampler`的位置，也可以设定`batch_size`等参数设定每个epoch的输出。

- world size决定把整个`Dataset`切成几份，而rank决定sampler输出的`indices`的起点（在原Dataset中的起点）

- 首先看源码：

  ```python
  import math
  import torch
  from . import Sampler
  import torch.distributed as dist
  
  
  class DistributedSampler(Sampler):
      """Sampler that restricts data loading to a subset of the dataset.
  
      It is especially useful in conjunction with
      :class:`torch.nn.parallel.DistributedDataParallel`. In such case, each
      process can pass a DitributedSampler instance as a DataLoader sampler,
      and load a subset of the original dataset that is exclusive to it.
  
      .. note::
          Dataset is assumed to be of constant size.
  
      Arguments:
          dataset: Dataset used for sampling.
          num_replicas (optional): Number of processes participating in
              distributed training.
          rank (optional): Rank of the current process within num_replicas.
          shuffle (optional): If true (default), sampler will shuffle the indices
      """
  
      def __init__(self, dataset, num_replicas=None, rank=None, shuffle=True):
          if num_replicas is None:
              if not dist.is_available():
                  raise RuntimeError("Requires distributed package to be available")
              num_replicas = dist.get_world_size()
          if rank is None:
              if not dist.is_available():
                  raise RuntimeError("Requires distributed package to be available")
              rank = dist.get_rank()
          self.dataset = dataset
          self.num_replicas = num_replicas
          self.rank = rank
          self.epoch = 0
          self.num_samples = int(math.ceil(len(self.dataset) * 1.0 / self.num_replicas))
          self.total_size = self.num_samples * self.num_replicas
          self.shuffle = shuffle
  
      def __iter__(self):
          # deterministically shuffle based on epoch
          g = torch.Generator()
          g.manual_seed(self.epoch)
          if self.shuffle:
              indices = torch.randperm(len(self.dataset), generator=g).tolist()
          else:
              indices = list(range(len(self.dataset)))
  
  
          # add extra samples to make it evenly divisible
          indices += indices[:(self.total_size - len(indices))]
          assert len(indices) == self.total_size
  
          # subsample
          indices = indices[self.rank:self.total_size:self.num_replicas]
          assert len(indices) == self.num_samples
  
          return iter(indices)
  
      def __len__(self):
          return self.num_samples
  
      def set_epoch(self, epoch):
          self.epoch = epoch
  
  ```

  

  上面有几个关键点：

  1. ` self.num_samples = int(math.ceil(len(self.dataset) * 1.0 / self.num_replicas))` 因为用的是`ceil`，所以能够保证所有数据都遍历到，但是也会导致可能需要 **从原Dataset的index的头**取indices补足。
  2. `self.total_size = self.num_samples * self.num_replicas`指的是所有进程加起来的总的samples数，可以看到可能会比原来的`Dataset`多
  3. `g.manual_seed(self.epoch)`保证了如果要shuffle，则shuffle出来的顺序会根据epoch数变化，所以具体使用时需要`train_sampler.set_epoch(epoch)`
  4. `indices += indices[:(self.total_size - len(indices))]`作用是把indices补全到需要的数目为止
  5. `indices = indices[self.rank:self.total_size:self.num_replicas]`是跳着取的，所以能够保证 **各个subset之间都是互不重叠的**



## DataParallel和DistributedDataParallel的区别

参考 [Comparison between `DataParallel` and `DistributedDataParallel`](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html#comparison-between-dataparallel-and-distributeddataparallel )

- DataParallel 默认是 **单进程，多线程的**，而DistributedDataParallel(DDP) 默认是应该更快的
- 如果模型过大导致没办法在 **one gpu** 上训练，并且数据也过大导致没办法在 **one machine**加载进来，可以同时用model parallel 和 DDP，前者对应把model 分开



## `torch.distributed.init_method`的三种初始化方式

参考：

[Initialization method]( https://pytorch.org/tutorials/intermediate/dist_tuto.html#initialization-methods )

[Initialization]( https://pytorch.org/docs/stable/distributed.html#initialization )

### 通过TCP通讯的方式：

- 需要提供master的ip和访问的port，需要注意 **这两个应该保证每一个rank（每一个进程或者每一个machine）都能访问到**
- 如果是单机多卡，那么把ip直接设为localhost(`127.0.0.1`)即可

### Shared file-system initialization

- 这个更加简单，直接用一个大家都能访问到的文件做共享即可
- 需要注意，这个文件 **最好是没有被创建过的，或者是每次训练完后都手动删除（虽然`init_process_group()`也会自动删除）**



### 通过环境变量

- **最最最最推荐这个，配合`torch.distributed.launch`，具体设置方法参考[torch.distributed.launch设置](### 法2：用`torch.distributed.launch`辅助开启进程)**

- 这个也是默认的方式，即默认传入`init_process-group`的方法是`env://`即可，rank和world size都通过环境变量读取

