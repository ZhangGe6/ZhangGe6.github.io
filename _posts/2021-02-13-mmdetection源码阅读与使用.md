---
title: mmdetection源码阅读与使用
author: Zhang Ge
date: 2021-02-13 17:00:00 +0800
categories: [实践, 框架学习]
tags: [mmlab]
math: true
---

# 官方Links

[mmdetection github repo](https://github.com/open-mmlab/mmdetection)



[mmdetection document](https://mmdetection.readthedocs.io/en/latest/tutorials/)



# 源码解读

## OpenMMLab 官方知乎专栏 :+1:

:one: ​[轻松掌握 MMDetection 整体构建流程(一)](https://zhuanlan.zhihu.com/p/337375549)



:two: [轻松掌握 MMDetection 整体构建流程(二)](https://zhuanlan.zhihu.com/p/341954021)



:three: ​[轻松掌握 MMDetection 中 Head 流程](https://zhuanlan.zhihu.com/p/343433169)



:four: [轻松掌握 MMDetection 中常用算法(一)：RetinaNet 及配置详解](https://zhuanlan.zhihu.com/p/346198300)

## 其他解读

[解构MMDetection](https://zhuanlan.zhihu.com/p/150144563)

### [一个知乎专栏-目标检测框架mmdetection入门](https://www.zhihu.com/column/c_1159526395804725248)


# 自己的理解和梳理

## 20210214

`mmdet/models/`解耦实现了网络架构的各个组件，如backbones、necks、heads、losses等；`configs/`含有多种配置文件，通过对应的配置文件，对`mmdet/models/`中定义的组件实例化。

>  追踪代码，可以看到，该实例化最终是通过`mmcv/utils/registry.py`的`build_from_cfg()` 方法实现的(可参见[解构MMDetection](https://zhuanlan.zhihu.com/p/150144563)中的分析)。



`configs/`配置文件具有继承机制。`__base__`中实现了`datasets`，`models`,` schedules`的“基配置”，其余配置文件可在这些“基配置”的基础上继承和覆写来完成，也可从头实现。继承的方法是在文件开头写上`__base__ = [../__base__/<some configs>]`，可以同时继承多个文件，并做对应的覆写。如`retinanet/retinanet_r50_fpn_1x_coco.py`中

```python
_base_ = [
    '../_base_/models/retinanet_r50_fpn.py',
    '../_base_/datasets/coco_detection.py',
    '../_base_/schedules/schedule_1x.py', '../_base_/default_runtime.py'
]
# optimizer
optimizer = dict(type='SGD', lr=0.01, momentum=0.9, weight_decay=0.0001)
```



> 在这样的继承机制里，子类中出现与父类相同的属性发生覆写，~~出现父类中没有的属性则为子类的新属性~~。另外，对于本身是字典的属性，子类中再次出现时，与普通属性不同的覆写行为不同，也呈现出一种继承的表现。比如：

父类中定义：

```python
    backbone=dict(
        type='ResNet',
        depth=50,
        num_stages=4,
        out_indices=(0, 1, 2, 3),
        frozen_stages=1,
        norm_cfg=dict(type='BN', requires_grad=True),
        norm_eval=True,
        style='pytorch'),
```

子类中再次出现

```python
    backbone=dict(
        norm_cfg=dict(requires_grad=False), norm_eval=True, style='caffe')
```

则最终的结果是

```python
    backbone=dict(
        type='ResNet',
        depth=50,
        num_stages=4,
        out_indices=(0, 1, 2, 3),
        frozen_stages=1,
        norm_cfg=dict(type='BN', requires_grad=False),
        norm_eval=True,
        style='caffe')
```





## 20210215

对一个完整的模型训练进行梳理



# 拾遗

[目标检测(MMdetection)-HOOK机制](https://zhuanlan.zhihu.com/p/238130913)



[Python 函数装饰器](https://www.runoob.com/w3cnote/python-func-decorators.html)