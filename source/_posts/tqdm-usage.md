---
title: tqdm使用
date: 2019-10-30 13:18:30
tags: tqdm, python
categories: python
---

## Basic
- 最直接的就是用tqdm把一个itereble object wrap起来，然后直接用：
    ```python
    with tqdm(range(config['INCV_epochs']), desc='INCV iter:{}'.format(kwargs['INCV_iter']), unit='epoch') as t:
        for epoch in t:
    ```

- 或者是自己手动update：
    ```python
    with tqdm(total=10, bar_format="{postfix[0]} {postfix[1][value]:>8.2g}",
            postfix=["Batch", dict(value=0)]) as t:
        for i in range(10):
            sleep(0.1)
            t.postfix[1]["value"] = i / 2
            t.update()
    ```

## `set_postfix()`, `set_description()`的用法
- `set_postfix()`默认情况下能够在进度条的最后显示一些参数，比如learning rate等
- `set_description()`则能够改变进度条前面的描述
- `set_postfix()`的输入应该是一个dict，或者kwargs，反正最后会转成那样的str，因此如果想自己控制str，应该用 `set_postfix_str()`
- **建议少用`write`，多用`set_posfix`/`set_postfix_str`。否则容易多行显示而非单行的progressbar**

## 不能单行显示的解决方法：
参考

[tqdm不能单行显示](https://www.zhihu.com/question/63371938)

[tqdm模块无法单行打印进度条](https://blog.csdn.net/weixin_41712499/article/details/92834110)

### 法1

```python
for i in tqdm(list, position=0, leave=True):
```

### 法2

```python
try:
    with tqdm(...) as t:
        for i in t:
            ...
except KeyboardInterrupt:
    t.close()
    raise
t.close()
```

