---
title: OpenCV基础使用
date: 2020-01-16 10:42:11
tags: 
- OpenCV
- python
categories:
- python
---

## OpenCV中的坐标
默认是图片（视频帧）的左上角为原点(0, 0)，width对应x轴，height对应y轴，→为正x轴，↓为正y轴

## 图片裁剪
直接用slicing即可，注意是`[y, x]`的表示方式，即:



```python
img[y_min:y_max, x_min:x_man]
```

默认是返回对应大小的图片 $H \times W \times C$

但若裁剪的范围超过了原有图片大小,**则返回$0 \times C$ 的结果, 强制保存这样的输出结果会出错**

## 基于VideoCapture对视频的常用操作




