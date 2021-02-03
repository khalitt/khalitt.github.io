---
title: Attention Models in Graphs A Survey
date: 2019-09-11 09:22:45
tag: attention
categories: paper

---



# Attention Models in Graphs A Survey

# 关键点
- 提及了三种分类方法：根据图的类型来看，其中大部分方法还是聚焦到homogeneous graph上，而heterogeneous graph的研究较少
- 三种attention的方法：

1. learnable attention weights

主要代表就是GAT（基于softmax使得所有的weights和为1，其中$a$是可学习的参数，也即attention)：

$$
\alpha_{0, j}=\frac{\exp \left(\operatorname{LeakyReLU}\left(\mathrm{a}\left[\mathrm{W}_{\mathrm{X}_{0}} \| \mathrm{W}_{\mathrm{X}_{j}}\right]\right)\right)}{\sum_{k \in \mathrm{F}_{\nu_{0}}} \exp \left(\operatorname{LeakyReLU}\left(\mathrm{a}\left[\mathrm{W} \mathrm{x}_{0} \| \mathrm{W} \mathrm{x}_{k}\right]\right)\right)}
$$


2. Similarity-based attention 

以AGNN 为代表，这种类型与上面1的最大的不同是，它直接地体现了“学习更加相似的embeddings" The difference is that the model explicitly learns similar hidden embeddings for objects that are relevant to each other since attention is based on similarity or alignment. 

$$
\alpha_{0, j}=\frac{\exp \left(\beta \cdot \cos \left(\mathbf{W} \mathbf{x}_{0}, \mathbf{W} \mathbf{x}_{j}\right)\right)}{\sum_{k \in \Gamma_{v_{0}}} \exp \left(\beta \cdot \cos \left(\mathbf{W} \mathbf{x}_{0}, \mathbf{W} \mathbf{x}_{k}\right)\right)}
$$

where $\beta$  is a trainable bias and “cos” represents cosine-similarity


3. Attention-guided walk

就是本文作者的文章GAM，主要思想就是用RNN encode 一条path/walk(看成subgraph），得到$\mathbf{h}_{t} \in \mathbb{R}^{h}$（即t时刻这条路线走过的所有点的信息都包含在内了），然后attention 函数$f^{\prime} : \mathbb{R}^{h} \rightarrow \mathbb{R}^{k}$把这个映射到k维，即k types of nodes，然后根据这个决定path/walk的下一步应该往哪走：如下图(b)所示


