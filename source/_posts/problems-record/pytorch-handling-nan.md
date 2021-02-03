---
title: pytorch nan处理
typora-root-url: pytorch-handling-nan
typora-copy-images-to: pytorch-handling-nan
date: 2021-02-01 20:36:37
tags:
- pytorch
categories:
- pytorch
---



# 问题

训练到中途出现Nan

![image-20210201204028670](/image-20210201204028670.png)



# 原因

暂时未知，按照 [Weights become NaN values after first batch step](https://discuss.pytorch.org/t/weights-become-nan-values-after-first-batch-step/87594)中提到的方法：

* `torch.autograd.set_detect_anomaly(True)`

* ```python
  for name, param in model.named_parameters():
      print(name, torch.isfinite(param.grad).all())
  ```



好像在发生问题之前都是正常的：

![image-20210201204550453](/image-20210201204550453.png)



![image-20210201204701789](/image-20210201204701789.png)





# 解决方法

基本无解，只能做一个异常捕捉的处理

```python
            loss = (w_loss + eps_loss) / 2. + un_loss + penalty
            if torch.isnan(loss):
                raise ValueError(f"Found invalid nan trains at batch {batch_idx} / epoch {epoch}")
```

```python
        try:
            res = trainer.train()
        except ValueError as e:
            logger.warning(f"{e}, return 0.0 res")
            break

        if res is not None:
            all_res.append(res)
```

