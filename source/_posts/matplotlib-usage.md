---
title: matplotlib使用
date: 2020-05-06 11:50:37
tags: 
- matplotlib
- python
categories: python
---

## 修改matplotlib图片大小

```python
%matplotlib inline
import matplotlib.pyplot as plt
plt.rcParams["figure.figsize"] = (6, 4)
plt.rcParams['figure.dpi'] = 300
```



## subplot在外面的legend

[How To Put Legend outside of Axes Properly in Matplotlib?](https://jdhao.github.io/2018/01/23/matplotlib-legend-outside-of-axes/)

```python
ax.legend(loc='lower left', bbox_to_anchor= (0.0, 1.01), ncol=2,
            borderaxespad=0, frameon=False)
```

![image-20200506162929703](../../../../../Library/Application Support/typora-user-images/image-20200506162929703.png)

## 直方图上添加value

参照https://stackoverflow.com/a/28931750



1. 先获得直方图的每个bins的值

   ```python
   click_hist_res=np.histogram(pd_all_uid_click_show_cleaned['ctr_per_user'], [0.0, 0.1, 0.2, 0.3,0.4,0.5,0.6,0.7,0.8,0.9,1.0])
   # 返回时两个东西，分别是value和区间
   ```

2. 获取rects，修改就可

   ```python
   fig=plt.figure(figsize=(12,8),dpi=300)
   ax=fig.add_subplot(111)
   y=pd_all_uid_click_show_cleaned['ctr_per_user']
   ax.hist(y, bins=[0.0, 0.1, 0.2, 0.3,0.4,0.5,0.6,0.7,0.8,0.9,1.0], align='mid')
   rects = ax.patches
   
   # Make some labels on bin.
   labels = [str(x) for x in click_hist_res[0]]
   
   for rect, label in zip(rects, labels):
       height = rect.get_height()
       ax.text(rect.get_x() + rect.get_width() / 2, height + 5, label,
               ha='center', va='bottom')
   plt.show()
   ```

   

