---
title: 使用python单例模式时要注意的key
typora-root-url: python-singleton-key
typora-copy-images-to: python-singleton-key
date: 2021-02-07 16:51:15
tags:
- python
categories:
- python
---



# 问题

训练时，只希望所有数据只有一份存储在内存中，即对于同样的一份数据只读取一次，但是又希望被多个不同的`dataset`或者`dataloader`共用，自然是需要单例模式。



一个经典的实现就是：

```python
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls_key not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]

    def clear(cls):
        # MyClass.clear() to clear
        cls._instances = {}
```

但是这种实现**仅仅只能保证一个类只产生一个实例，但是对于想用一个类来初始化所有的数据来说，并不可行，等价于初始化完train，val和test都不会初始化了**



很自然能够想到直接使用下面这种，**用`dataset_type`作为key来初始化**

```python
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        # Might vary based on order in args
        cls_key = args[1]
        if cls_key not in cls._instances:
            cls._instances[cls_key] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls_key]

    def clear(cls):
        # MyClass.clear() to clear
        cls._instances = {}

# 继承上面的单例
class XRMBData(metaclass=Singleton):
    def __init__(self, root, dataset_type, noise_type, noise_rate, seed, modalities=[0, 1]):
        print(f"XRMBData {dataset_type} was called")
        folder = 'seed_{:d}'.format(seed)
        label_folder = folder + '/labels/{}_{:.1f}'.format(noise_type, noise_rate)
        self.data = []
        self.clean_labels = []

        if dataset_type == 'test':
            self.labels = torch.from_numpy(np.load(os.path.join(root, label_folder, 'y_test.npy'))).long()
        else:  # val / train
            self.labels = torch.from_numpy(
                np.load(os.path.join(root, label_folder, 'noisy_y_{}.npy'.format(dataset_type)))).long()
            self.clean_labels = torch.from_numpy(
                np.load(os.path.join(root, label_folder, 'clean_y_{}.npy'.format(dataset_type)))).long()

        for md in modalities:
            md_mat = np.load(os.path.join(root, folder, 'md_{}/x_{}.npy'.format(md, dataset_type)))
            # Expect float otherwise be converted to Double
            self.data.append(torch.from_numpy(md_mat).float())

        self.dataset_type = dataset_type
```



但是注意到，这种实现只检查了`dataset_type`，如果说希望一次性跑多个`split`，那么上面这种处理就导致只有第一个split的数据被读入，之后的split都会用`split_0`的取代。



# 解决方法

主要有两种，推荐第一种：

* **把split信息也添加进`cls_key`**
* 每个`split`跑完之后都用`clear_instances`清除单例



对于第一种方法：

```python
class Singleton(type):
    _instances = {}

    def __call__(cls, *args, **kwargs):
        # Might vary based on order in args
        cls_key = ''.join([args[0], args[1]])
        # cls_key = args[1]
        if cls_key not in cls._instances:
            cls._instances[cls_key] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls_key]

    def clear(cls):
        # MyClass.clear() to clear
        cls._instances = {}
```





对比修改前后：

```bash
XRMBData train was called
train dataset with 329312 samples
Original fast dataloader finish with:7.8423 second
train dataset with 329312 samples
Original fast dataloader finish with:0.7790 second
train dataset with 329312 samples
Original fast dataloader finish with:0.7690 second
train dataset with 329312 samples
Original fast dataloader finish with:0.7571 second
train dataset with 329312 samples
Original fast dataloader finish with:0.7515 second
```



```bash
XRMBData train was called
train dataset with 329312 samples
1st split 1st epoch 1st batch used 6.5541 seconds
1st split 1st epoch used 6.7643 seconds
Original fast dataloader finish with:7.2792 second
XRMBData train was called
train dataset with 329312 samples
Original fast dataloader finish with:0.6898 second
XRMBData train was called
train dataset with 329312 samples
Original fast dataloader finish with:0.7306 second
XRMBData train was called
train dataset with 329312 samples
Original fast dataloader finish with:0.6899 second
XRMBData train was called
train dataset with 329312 samples
Original fast dataloader finish with:0.6729 second

```

