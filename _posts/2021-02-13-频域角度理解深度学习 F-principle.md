---
title: 频域角度理解深度学习 F-principle
author: Zhang Ge
date: 2021-02-13 22:00:00 +0800
categories: [专业积累, 基础知识]
tags: []
math: true
---

# \\(F-Principle\\)定理：

DNN倾向于按**从低频到高频的顺序**来拟合训练数据。

# 实验

![](/assets/img/20210213/T_domain.gif){: width="320" .left}
**Spatial Domain**

Red: the target function;

Blue: DNN output.

Abscissa: input;

Ordinate: output.
<br>

<br>

![](/assets/img/20210213/F_domain.gif){: width="320" .left}
**Fourier Domain**

Red: FFT of the target function;

Blue: FFT of DNN output.

Abscissa: frequency;

Ordinate: amplitude.
<br>

<br>

从上述实验[(图源)](https://ins.sjtu.edu.cn/people/xuzhiqin/fprinciple/index.html)可以看出，模型的拟合是有顺序的，首先从低频开始，逐渐转移至更高频率的拟合。



# 启发

从\\(F-principle\\)角度来理解过拟合现象：神经网络的泛化性能来源于它在训练过程，会更多关注低频分量。随着训练的进行，模型对训练集的拟合逐渐转化至高频成分，即对高频成分的拟合越来越好。但高频成分往往是噪声信号，因此导致了模型的泛化能力减弱。因此，提前停止训练（early-stopping）就能在实践中提高 DNN 的泛化能力。

![](/assets/img/20210213/early_stopping.png)

<center>early stopping的必要性</center>

# 参考

[知乎-如何从频域的角度解释 CNN（卷积神经网络）？](https://www.zhihu.com/question/59532432)



[Bilibli-许志钦-数学学院本科课程：统计计算与机器学习3 Frequency Principle](https://www.bilibili.com/video/BV1jE411u7G1?p=2)



[F-principle主页](https://ins.sjtu.edu.cn/people/xuzhiqin/fprinciple/index.html)