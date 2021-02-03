---
title: sklearn使用
date: 2019-09-18 09:41:10
tags: sklearn
categories: sklearn
---



## equal percentage切分数据集（Cross-Validation）

### sklearn.model_selection.StratifiedKFold

- 单纯的按比例切分，**能够保证不重复的folds**

### sklearn.model_selection.StratifiedShuffleSplit

- StratifiedKFold与RandomSplit的结合，因此 **无法保证所有folds都不一样**

### sklearn.model_selection.ShuffleSplit

- 单纯的随机切分

### sklearn.model_selection.RepeatedStratifiedKFold

- fold中的元素允许重复，**但是仍旧是保证各个fold都互不相同**

