---
title: 距离distance的一些理解
date: 2019-11-24 14:06:58
tags:
categories:
---


## euclidean distance
- euclidean distance 并不是 similarity，如果要normalize的的话，可以考虑:
    1. z变换，或者最直接的，x-min/max-min
    2. F=1 − exp(−x/λ) where λ is the average distance and x is the distance of the point you are evaluating.
- euclidean distance转化为similarity的方法其实有不少，具体选用哪个应该根据自己的问题来确定，比如如下的就可以考虑：
    1. $dist=1−sim, dist=1−simsim, dist=\sqrt{1−sim},   dist=−\log(sim)$
    2. $\frac{1}{1+d(p_1, p_2)}$

- 其实这些就是kernel，因此具体搜索的时候可以检索machine learning kernel来看
- 真的要完全理解，可以查阅这本书 [Encyclopedia of Distances](http://www.uco.es/users/ma1fegan/Comunes/asignaturas/vision/Encyclopedia-of-distances-2009.pdf)


