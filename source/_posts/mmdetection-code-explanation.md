---
title: mmdetection源码解析
date: 2019-12-04 17:50:41
tags: 
- mmdetection
- pytorch
categories: mmdetection
---

## train源码解析
train部分有两个train.py文件，一个位于tools下，另一个位于mmdet/api下，tools下的是更高层的api，mmdet/api下的主要是定义了`train_detector`。二者的关系是：

tools/train.py中调用mmdet/core/train.py中定义的`train_detector`。而`train_detector`又主要功能就是判断是多GPU分布式训练还是单GPU训练。

同时mmdet/core/train.py中还定义好了`batch_processor`以及它所调用的`parse_losses`，二者的作用如下：
- `batch_processor`就是执行一个batch的操作，在目标检测中对应某一个batch（2~3张imgs per gpu)，以train为例，则`batch_processor`就是执行一个forward并把所有输出记录再outputs中
- `parse_losses`，把outputs中的各个loss简单sum起来存到loss变量中，为后续回传梯度用。同时还把具体的value与key提取出来存到log_var的供logging用

下面着重以`_non_dist_train`为主

### `_non_dist_train` 主要用了Runner

1. 构建Dataloader（DataLoader还是用的pytorch的Dataloader）
2. 把model用mmdet定义的`MMDataParallel`包装起来
3. 引入mmcv中的helper类：runner。runner是训练的主要执行者。

### Runner：负责整个train或者test运行的过程

#### Runner的初始化

1. 需要确定model，batch_processor, optimizer等等，其中batch_processor主要是处理一个batch的。

2. 还需要注册hook（train和test的hook分开注册）

#### Runner的Hook
默认都是定义在mmcv/runner/hooks下面。

默认情况下，runner的`register_training_hooks`会定义

1. `LrUpdateHook`：控制学习率是如何更新的，必选
2. `OptimizerHook`控制如何更新模型
3. `LoggerHook`如何logging
   
还有`CheckpointHook`和`IterTimeHook`，作用如起字面意思。

特点：
- 除了lr和logging在runner中有专门的register函数，其余均共享`register_hook`函数定义
- 可以按照基类Hook的定义自定义Hook，要注意所有Hook具体执行任务的函数，基本都有一个runner这个选项，**因此需要在hook之间传递或者需要改变train或者test过程时，所有变量应该都储存在Runner中**

#### Runner的__dict__
主要有：
1. model
2. datasets
3. _max_iter 最大的执行步数
4. max_epoch 最大epoch数
5. _inner_iter 当前epoch下执行了多少个step
6. _iter 总共执行了多少个step
7. _epoch 当前epoch

可以看到runner主要还是保存了train或者test所需的基本变量，并记录训练过程以供所有hook一起使用。

#### Runner.run() 执行train或者test
定义在cfg文件中的workflow在此处具体执行。

对于workflow中的子flow，需要提前在Runner中定义好对应的成员函数

当前mmdet暂时**只支持一个workflow中为单train或者单test**，而没办法两个嵌套起来，因此test需要单独操作。


## two stage detector的训练过程

下面主要以fasterrcnn为例。可参考 [一文读懂Faster RCNN](https://zhuanlan.zhihu.com/p/31426458)

fasterrcnn定义在mmdet/models/detectors下面，定义很简单，只是简单地继承two stage detector，并把一些参数固定。因此可以认为**two stage detector**就是faster rcnn

### 提取feature
这部分的module主要有cfg中的backbone和neck确定，neck之所以称之为neck就是因为**它是直接接在backbone后面**，充当backbone和rpn_head等head之间的桥梁。

如默认faster rcnn的config中用FPN作为neck，需要注意的参数就是最后的`num_outs`，直接决定了**feats的level**，即multi level指的是输出list的len

### 获取rpn_cls_score和rpn_bbox_pred
这二者就是rpn_head的forward的输出rpn_outs。

1. rpn_cls_score：List[per_level_cls_scores]，里面的每个元素，分别对应各个level的featmaps上的每一个pixel（每一个点）的分数，**主要是用来判断这个点对应的anchor是不是bbox**
2. rpn_bbox_pred：List[per_level_bbox_pred]，实际上应该是用来做boundary regression的输入，**也即对anchor进行精修的**。值得一提的是，boundary box regression总共有两次，这里是第一次

### 基于rpn_outs算rpn_loss

### 获取proposals
主要在`rpn_head.get_bboxes`中完成，这里面实际上同时做了两步：

1. 对于每一个level，基于feat map用对应的anchor generator获取anchors(对应的AnchorGenerator在初始化的时候已经定义好了)

2. 基于每个level的rpn_cls_socre与rpn_bbox_pred，输出对应proposal


proposal_list: List[per_level_proposals]，即每个level都要对应proposal_list的一个元素，具体为(min(nums_post, max_num), 5)的tensor
