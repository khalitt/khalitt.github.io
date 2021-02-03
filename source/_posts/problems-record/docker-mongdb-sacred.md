---
title: docker-mongodb-sacred
typora-root-url: docker-mongodb-sacred
typora-copy-images-to: docker-mongodb-sacred
date: 2021-01-19 15:17:46
tags: docker
categories: problems
---



# Exit(100)的问题

尝试用下述的命令启动MongoDB：

```bash
docker run --name mongo -p 27018:27018 -v /home/weitaotang/multimodal/pytorch_hydra_results_temp/mongo_data:/data/db mongo
```

![image-20210119152405533](/image-20210119152405533.png)



猜测应该是权限问题，根据 [Permission error with mongo when running docker](https://stackoverflow.com/a/44782245)：最佳的解决方法**是不挂载，后期单独导出即可**

但是这里还是想存到一个固定的地方，则这里只可以存在`/tmp`旗下的地方，不好。最好还是定期的export即可。