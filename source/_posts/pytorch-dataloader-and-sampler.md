---
title: pytorch中的Dataloader与Sampler(以及collate_fn)
date: 2019-09-17 22:25:42
tags:
- pytorch
- Datalodaer
- Sampler
categories: pytorch

---

参考：

[Pytorch中的数据加载艺术](http://studyai.com/article/11efc2bf)

[pytorch之dataloader深入剖析](https://www.cnblogs.com/ranjiewen/p/10128046.html)

## DataLoader

- 支持两种类型的Dataset：

  - Map-Style :`torch.utils.data.Dataset`

  - Iteratable: `torch.utils.data.IterableDataset`

  前者需要重点实现`__len__`以及`__getitem__`，后者重点实现`__iter__`(注意应该 **返回一个迭代器**)
  

### 可通过`Sampler`和`BatchSampler`定制输出的结果

- Sampler:

  与`shuffle`冲突，本质上就是确定 **batch从什么样的indices list中得到**，当对Sampler进行赋值后，默认的batch抽取顺序是按照Sampler的`__iter__`返回的indices list **按顺序逐个抽取**，直至达到一个batch size或者遍历完所有元素

- BatchSampler:

  对比Sampler更进一步，当指定了之后，和`sampler`, `batch_size`, `shuffle`等参数都冲突。最终Dataloader的返回，是按照BatchSampler的`__iter__`确定的，**因此BatchSampler 的`__iter__`应该返回一个由各个batch indices组成的iterator，其中各个batch indices应该完全确定**





## Sampler

### 对应DataLoader sampler参数的Sampler

- `__iter__`需要返回的是一整个batch index的list，一维的即可，可以继续通过batch size来调整每一个batch包含的数目，**相当于这个sampler指定了batch输出indices的顺序，因此shuffle不可用**。 
- `__iter__`**必须返回一个迭代器**，因此 **通常要用iter()**

- 例子，可以看到当设定`__iter__`只输出前7个元素的时候，最后只遍历了7个元素

  ```python
  class SimpleSampler(Sampler):
      def __init__(self, data_source, n=5):
          self.data_source=data_source
          self.n = n
      def __len__(self):
          return len(self.data_source)
      def __iter__(self):
          return iter(torch.arange(self.n).tolist())
  ld_ss = DataLoader(fake_dataset, sampler=SimpleSampler(fake_dataset, 7), batch_size=2) # fake_dataset has 10 elements in total
  for item in ld_ss:
      print(item)
  ```

  ```
  {'idx': tensor([0, 1]), 'data': tensor([ 0, 10])}
  {'idx': tensor([2, 3]), 'data': tensor([20, 30])}
  {'idx': tensor([4, 5]), 'data': tensor([40, 50])}
  {'idx': tensor([6]), 'data': tensor([60])}
  ```

### 对应DataLoader batch_sampler参数的Sampler

- 对比sampler参数对应的Sampler定制的空间更大，**`__iter__`完全把batch size和顺序都确定了**

- `__iter__`需要输出的是 **iterator of indices，如`iter([[1,2,3], [4,5,6]])`**

- 例子：可以看到按照123,456的顺序输出了两个batch

  ```python
  class BalancedBatchSampler(Sampler):
      def __init__(self, data_source, test=True):
          self.data_source=data_source
          self.test=test
      
      @property
      def num_samples(self):
          return len(self.data_source)
      
      def __len__(self):
          return self.num_samples
      
      def __iter__(self):
          if not self.test:
              return iter([torch.randperm(self.num_samples).tolist()])
          else:
              return iter([[1,2,3], [4,5,6]])
  bsplr = BalancedSampler(fake_dataset, test=True)
  ld_bs = DataLoader(fake_dataset, batch_sampler=BalancedBatchSampler(fake_dataset, test=True))
  for item in ld_bs:
      print(item)
  ```

  ```
  {'idx': tensor([1, 2, 3]), 'data': tensor([10, 20, 30])}
  {'idx': tensor([4, 5, 6]), 'data': tensor([40, 50, 60])}
  ```

  

### 使用

- 直接用sampler参数对应的Sampler即可（还可以继续更改batch size）

- 自定义Sampler需要继承`torch.utils.data.Sampler`，也不一定需要 **把data_source作为`__init__`的参数**，如下的代码也是能够正常工作的

  ```python
  class SimpleSampler(Sampler):
      def __init__(self, n=5):
          self.n = n
      def __len__(self):
          return self.n
      def __iter__(self):
          return iter(torch.arange(self.n).tolist())
  ld_ss = DataLoader(fake_dataset, sampler=SimpleSampler(7), batch_size=2)
  for item in ld_ss:
      print(item)
  ```

  ```
  {'idx': tensor([0, 1]), 'data': tensor([ 0, 10])}
  {'idx': tensor([2, 3]), 'data': tensor([20, 30])}
  {'idx': tensor([4, 5]), 'data': tensor([40, 50])}
  {'idx': tensor([6]), 'data': tensor([60])}
  ```

  

### collate_fn

参考

[Pytorch技巧1：DataLoader的collate_fn参数](https://blog.csdn.net/weixin_42028364/article/details/81675021)

[How to use collate_fn()](https://discuss.pytorch.org/t/how-to-use-collate-fn/27181)

- 默认的collate_fn应该是`collate_fn(batch)`，其中`batch`是一个list，由batch_index直接作用在dataset上获得的：

  ```python
  class _MapDatasetFetcher(_BaseDatasetFetcher):
      def __init__(self, dataset, auto_collation, collate_fn, drop_last):
          super(_MapDatasetFetcher, self).__init__(dataset, auto_collation, collate_fn, drop_last)
  
      def fetch(self, possibly_batched_index):
          if self.auto_collation:
              # Default way to obatin batch data, returning a list
              data = [self.dataset[idx] for idx in possibly_batched_index] 
          else:
              data = self.dataset[possibly_batched_index]
          return self.collate_fn(data)
  ```

  

- 因此重载的话也应该如此




## 获取Subset的两种方式

### 通过新建sampler获得
- 只要用sampler就行了，设置对应的 `__iter__` 来实现
- 注意shuffle要设置为False，因此这个方法有个缺点是 **要自己实现在 `__iter__` 中实现shuffle**(或者直接通过 `RandomSampler`)

### 直接通过 `torch.data.Subset` （推荐）

- 直接指定好indices即可。但需要注意的是，**Subset本质上只是一个映射，没有重新创建新的内存空间，如果需要创建新的内存空间，此法不行**


    ```python
    class Subset(Dataset):
    r"""
    Subset of a dataset at specified indices.

    Arguments:
        dataset (Dataset): The whole Dataset
        indices (sequence): Indices in the whole set selected for subset
    """
    def __init__(self, dataset, indices):
        self.dataset = dataset
        self.indices = indices

    def __getitem__(self, idx):
        return self.dataset[self.indices[idx]]

    def __len__(self):
        return len(self.indices)
    ```


### 直接通过`torch.utils.data.SubsetRandomSampler`
- 更直接，全部数据都能复用。
- 本质也只是一个映射，**因此任何没办法通过Subset直接修改原来的Dataset**
    
    ```python
    class SubsetRandomSampler(Sampler):
    r"""Samples elements randomly from a given list of indices, without replacement.

    Arguments:
        indices (sequence): a sequence of indices
    """

    def __init__(self, indices):
        self.indices = indices

    def __iter__(self):
        return (self.indices[i] for i in torch.randperm(len(self.indices)))

    def __len__(self):
        return len(self.indices)
    ```

## 官方实现的几个sample的理解

### SequentialSampler
顾名思义，就是直接把`datasource`**原来的顺序**，逐个输出

- 顺序不变
- 由`__iter__`生成的迭代器（即`iter()`作用之后的输出），每次这个iterator输出的都是**一个int,而非torch.tensor**，因此理论上只是用来作为indices使用

### RandomSampler
与上面一样同样是一个iterable object，通过`iter`得到iterator之后，用`next`的输出就是**仍然是一个int**，只是此时顺序是乱的

### SubsetRandomSampler
这个要重点与`RandomSampler`进行比较，首先比较二者的`__iter__()`部分的源码：

- RandomSampler:

    ```python
    def __iter__(self):
        n = len(self.data_source)
        if self.replacement:
            return iter(torch.randint(high=n, size=(self.num_samples,), dtype=torch.int64).tolist())
        return iter(torch.randperm(n).tolist())
    ```
- SubsetRandomSampler:

```python
def __iter__(self):
    return (self.indices[i] for i in torch.randperm(len(self.indices)))
```

可以看到，RandomSampler返回的是一个iterator（因为用`iter`取出来了），而SubsetRandomSampler则只是返回一个tuple，也就是就是说只是一个iterable，还要用别的东西变成iterator才能用next取元素。

虽然对于`for in`循环来说，二者表现出的效果是一样的。

### BatchSampler
它的实现有所不同，目的本质上是把**在前述Sampler的基础上以batch的形式进行输出**

因此：
1. BatchSampler的构造函数需要制定一个sampler对象（不是类！！！）
2. `__iter__`返回的是一个generator，而非一个iterator
3. 输出是一个`由int组成的list`


### 可能一些利用思路：
1. 场景一：可以通过Ramdom Sampler + Batch Sampler来实现对indices的batch（即把原来Dataloader的一部分工作抽出来单独使用），如

```python
In [92]: tt = BatchSampler(RandomSampler(torch.tensor([3,5,65,765,5])), 2, False)

In [93]: for i in tt:
    ...:     print(i)
    ...:
[2, 0]
[4, 3]
[1]

In [94]: for i in tt:
    ...:     print(i)
    ...:
[3, 0]
[1, 4]
[2]

In [95]: for i in tt:
    ...:     print(i)
    ...:
[4, 1]
[0, 3]
[2]

In [96]: for i in tt:
    ...:     print(i)
    ...:
[2, 0]
[4, 1]
[3]
```

可以看到上述每次的输出都是**不一样的！！！**
