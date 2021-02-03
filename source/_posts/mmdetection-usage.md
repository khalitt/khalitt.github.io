---
title: mmdetection的使用基础
date: 2019-11-25 16:43:37
tags: mmdetection
categories:
---

## 基础概念

### ratio：高/宽 h/w
如`ratio=[0.5, 1.0, 2.0]`，则表示


### scale：各边放大的倍数
如`scale=[8.]`，则对应的两条边都放大八倍，即最后的面积变为原来的64倍


## 获取经过MMDataParallel的模型的module
- `MMDataParallel`直接继承`torch.nn.parallel.DataParallel`，对于model，还需要用`.module`才能获得对应的model
- 获取到上面的module之后，**直接通过`.`获取对应的模块就好了**，比如：

    ```python
    runner.model.module.bbox_head # obtain bbox_head module of faster rcnn
    ```

