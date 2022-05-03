---
title: 关于Batch Normalization
author: Zhang Ge
date: 2021-02-09 14:09:00 +0800
categories: [专业积累, 基础知识]
tags: []
math: true
---



`Batch-Normalization (BN)` 通过使用当前batch数据的均值mean和方差variance对它们进行标准化处理，可以使得模型的训练更加**快速**和**稳定**。实践中会把BN一般放在非线性激活函数的上一层或者下一层。


#  原理

## 训练时

BN之所以称之为batch normalization，就是因为normalization是沿batch_size维度进行的。

设某一个神经元对一个`batch`内的$$n$$个样本的输出为分别为$$Z^{(i)}, i=1, \cdots, n$$，BN层通过计算

$$
\mu = \frac{1}{n}\sum_i Z^{(i)}, \  \  \ \delta = \frac{1}{n}\sum_i (Z^{(i)} - \mu)  \tag{1, 2}
$$


得到它们的均值 $$\mu$$ 和方差 $$\delta$$. 然后通过


$$
Z_{norm}^{(i)} = \frac{Z^{(i)} - \mu}{\sqrt{\delta^2 - \epsilon}}   \tag{3}
$$

将原来该神经元的输出（可能是任意分布）转化到均值为0，方差为1的标准正态分布上。如[图3](https://towardsdatascience.com/batch-normalization-in-3-levels-of-understanding-14c2da90a338#b93c)所示。

![](/assets/img/20210209/BNed.jpeg)

<center>图3 batch_size为b, 3个神经元输出的示例。对于每个神经元，在使用BN前输出(沿batch_size维度)分布各异，经过BN操作后均服从相同的分布——正态分布</center>

最后，再对$$Z_{norm}^{(i)}$$做一个线性变换


$$
\hat{Z} = \gamma \times Z_{norm}^{(i)} + \beta \tag{4}
$$

$$\gamma$$ 和 $$\beta$$ 是可学习的参数，可以在正态分布的基础上调整，选择其最优的正态分布。

上面的解释是基于神经元neuron的输出展开的，但是**对于2D CNN中卷积得到的特征图，batch normalization是如何作用的呢？**

![](/assets/img/20210314/BN_conv.png)

<center>图4 2D CNN下的BN操作方式示意</center>

如图4，就是将某一个channel对应的batchsize个维度为(H, W)的tensor求解出**一对均值方差**，对这batchsize个(H, W)的tensor，其**每一个pixel value**减去该均值并除以该方差。[参考](https://www.zhihu.com/question/38102762/answer/391649040)

> Q: bn 不是对batch维度做归一化嘛，为什么h,w维度也要做？
>
> A: 因为CNN中卷积核是共享的，所以一张特征图应该被看作一个"神经元"的输出，因此归一化的时候N,H,W应该放在一起归一化。[参考(评论区)](https://blog.csdn.net/liuxiao214/article/details/81037416)

注意，BN层操作是**channel-wise**的：一个channel对应一对均值和方差，一对$$\gamma$$ 和 $$\beta$$ 。这也是在Pytorch中定义[BatchNorm2d](https://pytorch.org/docs/stable/generated/torch.nn.BatchNorm2d.html)层时，需要传入的是channel_num的缘故。[参考](https://blog.csdn.net/qq_27261889/article/details/87284076)和[参考](https://blog.csdn.net/weixin_38314865/article/details/104327852)

## 测试时

训练时，BN层可以为每一个`batch`输入数据计算均值和方差，但是**测试时**，BN层使用的均值和方差不是根据输入样本计算，而是使用训练过程中计算得到的值，直接从(3)式开始计算。可以参考这个[demo code](https://github.com/Erlemar/cs231n_self/blob/master/assignment2/cs231n/layers.py#L116). 

# 效果

![](/assets/img/20210209/BN_res.png)

<center>图5   x n代表以之前最优学习率的n倍进行训练</center>

可以看到，使用BN可以容忍更高的学习率，更快更好地收敛。

# :raising_hand_woman:QA

**Q：经常见到的`mean = [0.485, 0.456, 0=406] ， std = [0.229, 0.224, 0.225]`是什么？**

**A：**前面的(0.485,0.456,0.406)表示均值，分别对应的是RGB三个通道；后面的(0.229,0.224,0.225)则表示的是标准差，这些值[<u>是在ImageNet数据集计算出来</u>的](https://stackoverflow.com/questions/58151507/why-pytorch-officially-use-mean-0-485-0-456-0-406-and-std-0-229-0-224-0-2)，所以很多人都使用它们。不过在有了BN之后，它们已经[不必要](https://www.zhihu.com/question/264952701/answer/289210867)了。



**Q：对于RGB图而言，均值不应该是接近于128吗？为什么会是接近于0.5的数呢？**

**A：** 这是因为使用了`pytorch`的[transforms.ToTensor()](https://pytorch.org/vision/stable/transforms.html#torchvision.transforms.ToTensor)对图片进行了归一化处理（另外对于PIL Image or numpy.ndarray类型的输入，还将尺寸由` (H x W x C) `转化为了 `(C x H x W)`）。如

```python
transform = transforms.Compose(
    [transforms.ToTensor(),
     transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))])
```



**Q：上面说BN层在训练和测试时使用方法不同，那么模型如何区分自己是在训练还是在测试呢？** 

**A：**使用`model.train()`或者`model.train(True)`将`model`设为训练状态；使用`model.eval()`或者`model.train(False)`将`model`设为测试状态。当状态变化时，目前只有`dropout`和`batchnorm`两种层会受到影响([参考](https://stackoverflow.com/questions/51433378/what-does-model-train-do-in-pytorch))。`batchnorm`层在不同状态下的表现如前所述；对于[`dropout`](https://pytorch.org/docs/stable/_modules/torch/nn/modules/dropout.html)层，当处在测试状态时，该层输出值等于输入值(`identity map`)，相当于`disabled`。

> Tips: `dropout(p)`在训练时有一个`scaled by factor`$$\frac{1}{1-p}$$操作，目的是使得某一神经元输出在测试时的期望=训练时的期望，更进一步解释可参考[这里](https://stats.stackexchange.com/questions/205932/dropout-scaling-the-activation-versus-inverting-the-dropout)



**Q：Batch Normalization的一些局限？**

**A：**因为测试时BN层使用的均值和方差是训练过程中计算得到的值，因此当测试集和训练集数据分布差别较大时，会得到不好的测试效果。



**Q：BN层应放在非线性激活函数前面还是后面？**

**A：**BN作者本意是为了通过BN层，使得任何的参数值，都可以用所期望的分布(应该就是指正态分布)来生成激活值，因此将BN层放在了非线性激活函数前面一层。后来很多网络结构，比如ResNet，mobilenet-v2也都沿用了这一准则。不过也有[工作](https://github.com/ducha-aiki/caffenet-benchmark/blob/master/batchnorm.md#bn----before-or-after-relu)表示将BN层放在了非线性激活函数后面一层会取得更好的效果，但是没有给出有力的解释。reddit上对于这一问题有个激烈的[讨论](https://www.reddit.com/r/MachineLearning/comments/67gonq/d_batch_normalization_before_or_after_relu/dgqaksn/)。

**Q：Group Normlization?**

**A：**

# 参考

[Batch normalization in 3 levels of understanding](https://towardsdatascience.com/batch-normalization-in-3-levels-of-understanding-14c2da90a338#b93c)

[pytorch 计算图像数据集的均值和标准差](https://my.oschina.net/u/4286839/blog/3407778)