---
title: pytorch不同Dataloader实现方式的比较
typora-root-url: comparision-different_loading_methods
typora-copy-images-to: comparision-different_loading_methods
date: 2021-01-21 15:15:16
tags: 
- python
- io
categories: python
---



# 概述

主要是两种实现方式：

* 同一个数据集，按照不同split的方式切分（比如5 Fold Validation），保存到本地，然后依次读入5种split
* 同一个数据集，全部一次性读入，然后按照不同的split的方式进行切分，按照迭代器或者生成器的方式返回切分后的dataset或者Dataloader



核心代码：

第一种方式

```python
class Reuters(BaseDataLoader):
    def __init__(self, root, batch_size, dataset_type, noise_type, noise_rate, seed, balance=True, shuffle=True,
                 modalities=[0, 1], num_workers=1):
        self.dataset = ReutersDataset(root, dataset_type, noise_type, noise_rate, seed, balance, modalities)
        logger.info('{} dataset with {} samples'.format(dataset_type, len(self.dataset)))

        super().__init__(self.dataset, batch_size, shuffle, 0.0, num_workers)
```





第二种方式：

```python
def get_reuters_cv_dataloader(og_data, batch_size, noise_type, noise_rate, n_classes, seed, val_test_split, n_splits=5,
                              deal_imbalance=True,
                              modalities=[0, 1], num_workers=0, shuffle=True):
    # List[Dataset]
    all_train_dataset, all_val_dataset, all_test_dataset = get_reuters_cv_dataset(og_data, noise_type, noise_rate,
                                                                                  n_classes, seed, val_test_split,
                                                                                  n_splits=n_splits,
                                                                                  deal_imbalance=deal_imbalance,
                                                                                  modalities=modalities)
    for i in range(n_splits):
        train_data_loader = BaseDataLoader(all_train_dataset[i], batch_size, shuffle, 0.0, num_workers)
        val_data_loader = BaseDataLoader(all_val_dataset[i], batch_size, shuffle, 0.0, num_workers)
        test_data_loader = BaseDataLoader(all_test_dataset[i], batch_size, shuffle, 0.0, num_workers)
        yield train_data_loader, val_data_loader, test_data_loader

```









# 比较

比较的方式：5种切分方式（5 Fold Validation），跑10次取平均，并输出每一次两种初始化方式的总时间

```python
import time

from omegaconf import OmegaConf

from srcs.data_loader.all_data_loaders import get_reuters_cv_dataloader
from srcs.utils import instantiate

config = OmegaConf.load(
    '/home/weitaotang/multimodal/hydra_templates/srcs/independent_utlis/test_different_loaders/reuters_all_cv_train.yaml')


def independent_init():
    begin = time.time()
    for i in range(config.n_splits):
        config.root = config.root.format(i)
        train_data_loader = instantiate(config.train)
        valid_data_loader = instantiate(config.val)
        test_data_loader = instantiate(config.test)
        # end = time.time()
        # print("Split {} used {:.3f}s".format(i, end - begin))
    end = time.time()
    print('Independent init used {:.3f} seconds'.format(end - begin))
    res = end - begin
    return res


def all_init():
    begin = time.time()
    cv_dataloader = get_reuters_cv_dataloader(config.og_data, config.batch_size, config.noise_type, config.noise_rate,
                                              config.n_classes, config.seed, config.val_test_split,
                                              config.n_splits, config.deal_imbalance, config.modalities,
                                              config.num_workers, config.shuffle)

    # cv_dataloader_iter = iter(cv_dataloader)
    # train, valid, test = next(cv_dataloader_iter)
    for i, (train, valid, test) in enumerate(cv_dataloader):
        end = time.time()
        # print("Split {} used {:.3f}s".format(i, end - begin))
        pass

    end = time.time()
    print('ALL init used {:.3f} seconds'.format(end - begin))
    res = end - begin
    return res


if __name__ == '__main__':
    all_res = [[], []]
    for _ in range(10):
        all_res[0].append(independent_init())
        all_res[1].append(all_init())
    print("Independent avg:{:.6f} | all avg: {:.6f}".format(sum(all_res[0]) * 1. / len(all_res[0]),
                                                            sum(all_res[1]) * 1. / len(all_res[1])))

```



# 结果

```bash
Independent init used 6.621 seconds
ALL init used 6.215 seconds
Independent init used 6.875 seconds
ALL init used 6.074 seconds
Independent init used 6.661 seconds
ALL init used 6.247 seconds
Independent init used 6.416 seconds
ALL init used 5.833 seconds
Independent init used 6.244 seconds
ALL init used 6.292 seconds
Independent init used 7.124 seconds
ALL init used 5.922 seconds
Independent init used 6.636 seconds
ALL init used 6.253 seconds
Independent init used 6.307 seconds
ALL init used 6.150 seconds
Independent init used 6.520 seconds
ALL init used 5.989 seconds
Independent init used 6.841 seconds
ALL init used 6.281 seconds
Independent avg:6.624528 | all avg: 6.125607

Process finished with exit code 0
```



可以看到总体上还是**方式二快一些**，**直接读入所有数据然后直接进行处理**。**但是弊端也比较明显**：

* 不够灵活，比如不想进行validation的时候，就需要另外再重新实现
* 占用内存可能较大，比如不想载入全部数据时候，等于浪费了内存



因此如果**不是密集io的情况，方式一就够了，更灵活**