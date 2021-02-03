---
title: "解决Pytorch RuntimeError: element 0 of tensors does not require grad and does not have a grad_fn"
typora-root-url: pytorch-does-not-require-grad-and-does-not-have-a-grad_fn
typora-copy-images-to: pytorch-does-not-require-grad-and-does-not-have-a-grad_fn
date: 2021-02-03 00:36:43
tags:
- pytorch
categories:
- pytorch
---



# 问题

使用IEG系列过程中，发生如下问题`RuntimeError: element 0 of tensors does not require grad and does not have a grad_fn`：

![image-20210203004302851](/image-20210203004302851.png)



# 原因

模型写错了！！要注意要用meta module！！不能常规的`nn.Module`，因为这里面返回的`params`都是`nn.Paramter`而非像`MetaModule`那样是`torch.tensor`

![image-20210203004956976](/image-20210203004956976.png)



# 解决方法

直接改回来就好了！！