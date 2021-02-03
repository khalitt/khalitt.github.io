---
title: DGL使用基础
date: 2019-09-16 16:02:54
tags:
categories:
---



## Builtin message passing functions

- message function只能用在arguments里面明确指出了`message_func`的函数中，因此若把一个builtin message function作为`apply_node`的arguments会报错：

  ```python
  func = fn.u_add_v('nf','t','e')
  g.apply_nodes(func)
  ```

  ```
  ---------------------------------------------------------------------------
  TypeError                                 Traceback (most recent call last)
  <ipython-input-16-629bfb5541ce> in <module>
  ----> 1 g.apply_nodes(func)
  
  ~/.conda/envs/ptpy_3/lib/python3.6/site-packages/dgl/graph.py in apply_nodes(self, func, v, inplace)
     2094                                            apply_func=func,
     2095                                            inplace=inplace)
  -> 2096             Runtime.run(prog)
     2097 
     2098     def apply_edges(self, func="default", edges=ALL, inplace=False):
  
  ~/.conda/envs/ptpy_3/lib/python3.6/site-packages/dgl/runtime/runtime.py in run(prog)
        9         for exe in prog.execs:
       10             # prog.pprint_exe(exe)
  ---> 11             exe.run()
  
  ~/.conda/envs/ptpy_3/lib/python3.6/site-packages/dgl/runtime/ir/executor.py in run(self)
      127         node_data = self.fdnode.data
      128         if self.fdmail is None:
  --> 129             udf_ret = fn_data(node_data)
      130         else:
      131             mail_data = self.fdmail.data
  
  ~/.conda/envs/ptpy_3/lib/python3.6/site-packages/dgl/runtime/scheduler.py in _afunc_wrapper(node_data)
      267     def _afunc_wrapper(node_data):
      268         nbatch = NodeBatch(graph, v, node_data)
  --> 269         return apply_func(nbatch)
      270     afunc = var.FUNC(_afunc_wrapper)
      271     applied_feat = ir.NODE_UDF(afunc, v_nf)
  
  TypeError: 'BinaryMessageFunction' object is not callable
  ```

- 使用builtin message function要注意dtype，如果是`torch.int64`会报错，但是`torch.float32`就不会（如u_add_v)

### src与dst的理解

![](https://raw.githubusercontent.com/khalitt/IMG_REPO/master/20190916170051.png)

- u 代表source，v代表destination。**直接用g.edges()查看，第一个向量对应u，第二个向量对应v（因为默认添加edge的时候是按照(u,v)的顺序添加，即从tuple中左边的node为source，右边的是destination**

- binary operation的arguments中的`lfs field`    `rhs field` 指的是那个符号（比如add，div，mul两边分别是什么，比如`u_add_v`指的是左边`lfs field`对应的是source，`rhs field`对应的是destinations。反过来`v_add_u`则指的是`lfs field`对应destinations，`rhs field`对应的是source

- 同理`u_add_e`与`e_add_u`的理解，但要注意的是，**这里面`lfs field` 与 `rhs field`不能随便互换，因为node features和edge features的维度一般不一样**

### send与recv机制的理解

- send需要一个message passing function（如u_add_v）, recv需要一个reduce function（如sum）。前者直接输出到mailbox中（既不在edata也不再ndata，无法直接访问），后者则直接输出到ndata中。

- mailbox 是储存在nodes的里面，使用完send之后是没办法直接**直接访问mailbox的，必须用recv()再输出到ndata中**
- update_all相当于把以上两个步骤合二为一，直接传入两个func即可。

## Node UDF, Edge UDF的理解

- 通常`message_func`要求Edge UDF（即参数为`EdgeBatch`），`reduce_func`为Node UDF（参数为`NodeBatch`)

### message_func(Edge UDF)

- 参数``EdgeBatch``是包含 **该graph全部edges的batch，因此只会执行一次**
- `EdgeBatch.src`是用graph.edges()[0]对应nodes得到的，`EdgeBatch.dst`是用graph.edges()[0]对应nodes得到的，他们各自包含对应顺序的data



### reduce_func(Node UDF)

- `nodes.mailbox`中的data为$N_{nodes} \times M_{dst}$ or $N \times \times M_{dst} \times dim_1 \times dim_2 \times ... \times dim_k$， 其中$N_{nodes}$ 表示的是这个node batch中所包含的节点数，$M_{dst}$表示那个node **在EdgeBatch.dst中对应edge的message，即首先得到graph.edges[1] 该节点所对应的edges idx，然后用这个index list去获得对应的edge的message（经自定义的message_func或builtin），最后抽出来作为这个node的message，并在最前面添加一个维度以便collate**

- 一个node batch中每个node的degree**应该是一样的**

- nodes.mailbox生成的思路是：

  1. **按照degree或者在dst index(graph.edges()[1])中出现的顺序，把可能同degree的nodes抽出来作为一个batch，每个node的message的维度若是$M_{dst}$ （表示每条edge的message为一个scalar） 或者$M_{dst} \times dim_1 \times dim_2 \times ... \times dim_k$**，

  2. 再在 **最前面加一个维度，并collate成**$N_{nodes} \times M_{dst}$ or $N \times \times M_{dst} \times dim_1 \times dim_2 \times ... \times dim_k$
  3. 把collate好的tensor输入到mailbox中对应的位置。

- `reduce_func`需要做的是 **把 $N_{nodes} \times M_{dst}$ 或者 $N \times \times M_{dst} \times dim_1 \times dim_2 \times ... \times dim_k$ 转化为 $N \times dim_{output}$ 的数据。**

- 需要注意mailbox中message，**第一个维度对应这个NodeBatch中的nodes index，第二个维度对应以该node为dst的edge的另一个节点，之后才是各条edge的对应的message**

- 当最后的结果为一个scalar的时候，**也要保留一个维度**， 即$N$，可通过`unsqueeze()`实现。

  如下面的例子sum **不能把dim留空，否则报错，至少要有一个维度。**：

  ```python
  def self_d_rduc(nodes):
      print('Nodes data:{}'.format(nodes.data))
      print('Message:{}'.format(nodes.mailbox['m']))
      r_msg = nodes.mailbox['m'].sum(-1)
      print('Output_message: {}'.format(r_msg))
      return {'m_out':r_msg}
      
  g.recv(reduce_func=self_d_rduc)
  ```

  ```
  Nodes data:<dgl.utils.LazyDict object at 0x7fff2b8c5a58>
  Message:tensor([[ 182.,  362.,  542.,  902., 1082., 1262., 1442., 1622.]])
  Output_message: tensor([7396.])
  ```

## dgl._ffi.base.DGLError：输入的点必须连续

从networkx graph生成的时候，**需要把原来不连续的node index变成连续的，从0开始的，为此需要做一个mapping(`nx.relabel_nodes` or `nx.convert_node_labels_to_integers)**，推荐后者，因为还可以把 **原来的nodes index放到nx graph的nodes attributes中**



## 把模型转移到gpu：`.to(torch.device('cuda'))`

直接`DGLGraph.to(torch.device('cuda'))`即可





## Multi GPU训练：应该用Distributed

参考：

[Multi_GPU GAT and Model_zoo](https://discuss.dgl.ai/t/multi-gpu-gat-and-model-zoo/434)

[Training GCN on multiple GPUs](https://github.com/dmlc/dgl/issues/838)

上面分别有两个例子，以及下面Pytorch官方的Distributed Training的教程

[When will multi-gpu be supported](https://github.com/dmlc/dgl/issues/410)

[ WRITING DISTRIBUTED APPLICATIONS WITH PYTORCH](https://pytorch.org/tutorials/intermediate/dist_tuto.html#writing-distributed-applications-with-pytorch)


## batch的方法

参考：
- [重点关注其中对Dataloader和自己写collate function的方法](https://docs.dgl.ai/tutorials/basics/4_batch.html)
- [model_zoo chem中用到的一个collate函数，可以参考](https://github.com/dmlc/dgl/blob/8b17a5c1d538df5342a6d0bc9d3dd198b6de3ce4/examples/pytorch/model_zoo/chem/property_prediction/utils.py#L216)

直接使用pytorch的Dataloader，**再加上自己写的`collate function`即可。**