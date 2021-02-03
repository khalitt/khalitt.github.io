---
title: hydra可重复性
typora-root-url: hydra-reproducibility
typora-copy-images-to: hydra-reproducibility
date: 2021-02-01 00:06:31
tags:
- hydra
- joblib
categories:
- hydra
---



# 问题

虽然设置了pytorch和numpy的seed，然而同样的参数，用`joblib`跑出来的实验结果仍然是不同的，但是奇怪的是如果不用`joblib`，默认的launcher同样的参数直接跑却没问题



# 原因

具体原因不详，猜测是和`joblib`有关。后通过实验验证：在`main()`外面设置的seed，在`joblib`的环境下不起效。



# 解决方法

## numpy

很简单，直接在`main()`函数内部设置`seed`就可以了



## pytorch

稍微有点麻烦：

* 如果不用gpu，即不设置CUDA相关的`deterministic`的行为，和numpy一样在`main`函数内部直接设置`seed`即可
* 但是如果要使用`gpu`，则大概率要设置CUDA相关的`deterministic`的参数，此时直接在`main()`内部设置`seed`将会直接报错：`TypeError: can't pickle CudnnModule objects`，后查阅资料，[Can't pickle CudnnModule objects](https://github.com/ray-project/ray/issues/8569#issuecomment-632993176)，发现ray也有这个问题，参考这个回答的做法就可以了



## 汇总

从原来的：

```python
import logging

import hydra
import numpy as np
import torch

logger = logging.getLogger(__name__)

SEED = 1234
np.random.seed(SEED)
torch.backends.cudnn.deterministic = True
torch.backends.cudnn.benchmark = False
torch.manual_seed(SEED)

@hydra.main(config_path='./', config_name='myconfig_multi')
def hydra_main(config):
    # logger.info(f"num iter:{config.num_iters} pytorch int:{torch.randint(10, (1,))}")
    logger.info(f"num iter:{config.num_iters} int:{np.random.randint(10)} pytorch int:{torch.randint(10, (1,)).item()}")


if __name__ == '__main__':
    hydra_main()

```



改成下面这种

```python
import logging

import hydra
import numpy as np
import torch

logger = logging.getLogger(__name__)

SEED = 1234


@hydra.main(config_path='./', config_name='myconfig_multi')
def hydra_main(config):
    import torch
    torch.manual_seed(SEED)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
    np.random.seed(SEED)
    # logger.info(f"num iter:{config.num_iters} pytorch int:{torch.randint(10, (1,))}")
    logger.info(f"num iter:{config.num_iters} int:{np.random.randint(10)} pytorch int:{torch.randint(10, (1,)).item()}")


if __name__ == '__main__':
    hydra_main()

```



后面脚本的效果

```bash
[2021-02-01 00:18:23,724][HYDRA] Joblib.Parallel(n_jobs=5,backend=loky,prefer=processes,require=None,verbose=0,timeout=None,pre_dispatch=2*n_jobs,batch_size=auto,temp_folder=None,max_nbytes=None,mmap_mode=r) is launching 10 jobs
[2021-02-01 00:18:23,725][HYDRA] Launching jobs, sweep output dir : multirun/2021-02-01/00-18-21
[2021-02-01 00:18:23,725][HYDRA]        #0 : num_iters=0
[2021-02-01 00:18:23,725][HYDRA]        #1 : num_iters=1
[2021-02-01 00:18:23,725][HYDRA]        #2 : num_iters=2
[2021-02-01 00:18:23,725][HYDRA]        #3 : num_iters=3
[2021-02-01 00:18:23,725][HYDRA]        #4 : num_iters=4
[2021-02-01 00:18:23,726][HYDRA]        #5 : num_iters=5
[2021-02-01 00:18:23,726][HYDRA]        #6 : num_iters=6
[2021-02-01 00:18:23,726][HYDRA]        #7 : num_iters=7
[2021-02-01 00:18:23,726][HYDRA]        #8 : num_iters=8
[2021-02-01 00:18:23,726][HYDRA]        #9 : num_iters=9
[2021-02-01 00:18:52,994][__main__][INFO] - num iter:0 int:3 pytorch int:2
[2021-02-01 00:18:53,089][__main__][INFO] - num iter:1 int:3 pytorch int:2
[2021-02-01 00:18:53,350][__main__][INFO] - num iter:2 int:3 pytorch int:2
[2021-02-01 00:18:54,099][__main__][INFO] - num iter:4 int:3 pytorch int:2
[2021-02-01 00:18:54,232][__main__][INFO] - num iter:3 int:3 pytorch int:2
[2021-02-01 00:18:54,610][__main__][INFO] - num iter:5 int:3 pytorch int:2
[2021-02-01 00:18:54,740][__main__][INFO] - num iter:6 int:3 pytorch int:2
[2021-02-01 00:18:54,860][__main__][INFO] - num iter:7 int:3 pytorch int:2
[2021-02-01 00:18:55,134][__main__][INFO] - num iter:8 int:3 pytorch int:2
[2021-02-01 00:18:55,147][__main__][INFO] - num iter:9 int:3 pytorch int:2

```

