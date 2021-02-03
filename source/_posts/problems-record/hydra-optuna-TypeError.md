---
title: "hydra optuna sweeper TypeError: 'float' object is not iterable的错误"
typora-root-url: hydra-optuna-TypeError
typora-copy-images-to: hydra-optuna-TypeError
date: 2021-01-20 16:42:06
tags: 
- hydra
- optuna
categories:
- optuna
---



# 问题

不论是直接跑 [hydra-optuna-sweeper](https://github.com/toshihikoyanase/hydra-optuna-sweeper)或者 [facebookresearch](https://github.com/facebookresearch)/**[hydra](https://github.com/facebookresearch/hydra)**的官方例子，都会报错`TypeError: 'float' object is not iterable`

![image-20210120164237711](/image-20210120164237711.png)



# 原因

optuna从2.4.0开始之后，要求主函数的返回的结果必须得是`List[]`类型的。这里出错的阶段是把运行结果`tell`给optuna的过程中，返回的结果是单一的float而不是list



# 解决方法

在返回结果中添加一个`list`即可

```python
from typing import Any

import hydra
from omegaconf import DictConfig


@hydra.main(config_path="conf", config_name="config")
def evaluate(cfg: DictConfig) -> Any:
    x = cfg.x
    y = cfg.y
    # return (x - 2) ** 2 + y # 报错的原版
    return [(x - 2) ** 2 + y]


if __name__ == "__main__":
    evaluate()

```

