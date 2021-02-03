---
title: pytorch-meta-learning
typora-root-url: pytorch-meta-learning
typora-copy-images-to: pytorch-meta-learning
date: 2021-01-08 18:18:41
tags: 
- pytorch
- meta learning
categories: pytorch
---



# Pytorch 在Meta Learning计算中的不足

正如 [higher](https://higher.readthedocs.io/en/latest/#higher-documentation) 首页所说

> `higher` is a library providing support for higher-order optimization, e.g. through unrolled first-order optimization loops, of “meta” aspects of these loops. It provides tools for turning existing torch.nn.Module instances “stateless”, **meaning that changes to the parameters thereof can be tracked**, **and gradient with regard to intermediate parameters can be taken**. It also provides a suite of differentiable optimizers, to facilitate the implementation of various meta-learning approaches.



即原版的`torch.nn.Module`不支持对parameters的梯度操作，导致形如像learning to reweight这一类方法用不起来



facebook自己的工程师在也有说明：

> Another issue you’re gonna face is that `nn.Parameter()` are built explicitly to be leaf Tensors (with no history) and so you won’t be able to change them with a Tensor you try to backprop through.
>
> You can check the [higher 2](https://github.com/facebookresearch/higher) package to do this properly (they solve this problem of nn.Parameter for you). Otherwise, if you want to do it manually, you can find some info in this thread: [How does one have the parameters of a model NOT BE LEAFS?](https://discuss.pytorch.org/t/how-does-one-have-the-parameters-of-a-model-not-be-leafs/70076/9)



# dirty、不用框架的方法

参考：

1.  [How does one have the parameters of a model NOT BE LEAFS?](https://discuss.pytorch.org/t/how-does-one-have-the-parameters-of-a-model-not-be-leafs/70076/9)  详细的讨论，**推荐仔细阅读**

**不推荐！！**



# 推荐方法：直接用higher框架

参考：

1. [Learning-to-Reweight-Examples-for-Robust-Deep-Learning-with-PyTorch-Higher](https://github.com/TinfoilHat0/Learning-to-Reweight-Examples-for-Robust-Deep-Learning-with-PyTorch-Higher) **直接参考其中的notebook！！推荐！！！**

