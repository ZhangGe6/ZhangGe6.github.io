---
title: tensorboard使用
author: Zhang Ge
date: 2021-11-16 19:45:00 +0800
categories: [实践, 框架学习]
tags: [tensorboard]
---
# 关于本页

tensorboard是一款经典的机器学习可视化工具。它原本是tensorflow的可视化工具，pytorch从1.2.0开始正式支持tensorboard。本页介绍我在使用过程中的一些操作和经验记录，便于日后参考复用，主要结合pytorch。

- pytorch提供的tensorboard官方文档和示例：[戳这里](https://pytorch.org/docs/stable/tensorboard.html)
- 一些已有的文章：[1](https://zhuanlan.zhihu.com/p/103630393)

# 一些操作记录

## 使用方法

要运行tensorboad，需要在代码中加入

```python
from torch.utils.tensorboard import SummaryWriter
writer = SummaryWriter()

writer.add_xxx()

writer.close()   # this can be ESSENTIAL, otherwise the last graph would not show!
```

生成的日志文件默认存放在`./runs`下。

> 可以在定义`writer`时使用`writer = SummeryWriter(log_dir=...)`来指定日志保存位置。

在终端中执行

```bash
tensorboard --logdir=runs
# for more usage, use `tensorboard -h` 
```

这时默认在`http://localhost:6006/ `打开tensorboard页面，显示可视化内容。

## 常用操作

### 绘制scalar曲线

```python
writer.add_scalar(tag, value, global_step)
```

若要把多张图像绘制在一张图上，则使用：[参考](https://pytorch.org/docs/stable/tensorboard.html#torch.utils.tensorboard.writer.SummaryWriter.add_scalars)

```python
writer.add_scalars(main_tag, tag_scalar_dict, global_step)
```



### 绘制直方图（如查看网络权重分布）

```python
for name, param in model.named_parameters():
    writer.add_histogram(name, param)
```

坑1：**到`HISTOGRAMS`栏查看直方图**（有一次我在`DISRUBUTIONS`栏看了半天愣是啥也看不到）

坑2：记得最后调用`writer.close()`，否则最后一层的权重分布图（我这边）是看不到的

### 显示matplotlib图像

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots() 
ax.plot(A, B)
ax.set_title('title')

writer.add_figure(
    'matplotlib demo', 
    fig
)
```

# 使用过程中遇到的问题及解决

## tensorboard: command not found

[解决方法](https://stackoverflow.com/a/53870810/10096987)

