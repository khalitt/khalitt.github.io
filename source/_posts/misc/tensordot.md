---
title: tensordot
typora-root-url: tensordot
typora-copy-images-to: tensordot
date: 2021-07-24 22:54:44
tags:
categories:
---

# tensordot的理解

1. 参考 [numpy的tensordot的说明文档](https://numpy.org/doc/stable/reference/generated/numpy.tensordot.html)，说的比较详细，但是依旧有点晦涩难懂



在tensorflow中有一样的函数，功能和numpy中的一样。



顾名思义，tensordot的含义就是“张量的点积“，具体到numpy或者tensorflow中的使用，有两种：

1. 如果给定的axes是一个int N，则直接第一个张量的最后N维度和第二个张量的前N个维度逐个元素相乘，然后加起来（注意这里的加，对应+，对于number就是数字相加，对于字符串等价于拼接）
2. 如果给第的axes是list，则其中每一个元素应该是一个tuple，tuple的第一个元素表示操作第一个张量对应的维度，第二个元素表示操作第二张量对应的维度



比如：a的维度是$(2,3,4)$, b的维度是$(3,4,5)$:

* `np.tensordot(a, b, 2)`的结果的维度应该是$(2,5)$​​​，具体的操作可以理解为，固定a的第一个维度和b的最后一个维度（依次取值，作为最外层的循环，即`i,j`），a的后两个和b的前两个维度相同位置的元素依次相乘，求和，作为对应位置的元素

```python
ans = np.zeros(2,5)
for i in range(2):
	for j in range(5):
		for k in range(3):
			for n in range(4):
				ans[i,j] += a[i,k,n] * a[k,n,j]
```



* `np.tensordot(a, b, (1, 0))`的结果的维度应该是$(2,4,4,5)$​​，具体操作可以理解为，固定`a[i,k,j]`和`b[n,f,g]`中的`i,j,f,g`维度（即对四个位置依次进行取值，作为最外层的循环），`k`和`n`这两个维度相同位置的元素依次进行相乘，求和

```python
ans = np.zeros(2,4,45)
for i in range(2):
	for j in range(5):
		for f in range(4):
			for g in range(5):
                for k in range(3):
                    for n in range(4):
                        ans[i,j,f,g] += a[i,k,j] * a[n,f,g]
```



* `np.tensordot(a, b, ((1, 2), (0,1))`的结果的维度应该是$(2,5)$​​​，具体操作可以理解为，固定`a[i,k,n]`和`b[k,n,j]`中的`i,j`维度（即对两个位置依次进行取值，作为最外层的循环），`k`和`n`这两个维度，在两个张量中，相同位置的元素依次进行相乘，并求和。实际上这就等同于第一种情况`np.tensordot(a,b,2)`





按照上述的理解，自然也就不难理解下面的操作了：

![image-20210724225721272](/image-20210724225721272.png)



特别注意上面的`n`和`k`的顺序：**是先4那个维度，再到3那个维度，因为`axes=[(1,0),[0,1]]`，所以最内层的循环是从`a[k, 0, i] * b[0, k, j]`开始** 

