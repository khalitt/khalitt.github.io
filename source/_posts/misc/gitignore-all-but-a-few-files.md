---
title: gitignore忽略整个文件夹下所有文件但保留一些
typora-root-url: gitignore-all-but-a-few-files
typora-copy-images-to: gitignore-all-but-a-few-files
date: 2021-08-20 22:42:32
tags: git
categories:
---



# 参考

[.gitignore设置跟踪忽略文件夹中文件](https://blog.csdn.net/u011475134/article/details/71521725)说的挺全的，可以好好参考



# 常见错误原因

假设想要忽略所有的名为Win7Release或release的文件夹下除.txt文件之外的文件，很容易写成如下错误形式

```bash
*[Rr]elease/
!*[Rr]elease/*.txt

```

注意：上面已经说过git对于.gitignore配置文件是按行从上到下进行规则匹配的，**由于先执行`*[Rr]elease/`忽略了所有符合条件的文件夹，接下来执行`!*[Rr]elease/*.txt`时找不到名为`*[Rr]elease`的文件夹，也就无法追踪这些文件夹下的txt文件了**。

所以，在这里，**第一步我们不应该忽略这些文件夹，而应该忽略这些文件夹下的所有文件**，正确规则添加如下


```bash
[Rr]elease/*
!*[Rr]elease/*.txt
!*[Rr]elease/files/
*[Rr]elease/files/*
!*[Rr]elease/files/*.txt

*/bin/*[Rr]elease/*
!*/bin/*[Rr]elease/*.txt
!*/bin/*[Rr]elease/files/
*/bin/*[Rr]elease/files/*
!*/bin/*[Rr]elease/files/*.txt
```

说明：
1. 执行`*[Rr]elease/*`，忽略根目录下符合条件的文件夹下的所有文件
2. 执行`!*[Rr]elease/*.txt`，追踪根目录下符合条件的文件夹下的txt文件
3. 执行`!*[Rr]elease/files/`，追踪根目录下符合条件的文件夹下的files文件夹
4. 执行`*[Rr]elease/files/*`，忽略files文件夹下的所有文件
5. 执行`!*[Rr]elease/files/*.txt`，追踪files文件夹下的txt文件



所以这样的做法是不对的，这样并不能忽略target文件下除了jar文件外的所有文件：

```bash
target
!target/*.jar
```



# 正确做法

```bash
target/*
!target/*.jar
```

![image-20210820225100993](/image-20210820225100993.png)

