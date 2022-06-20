---
title: matplotlib系统学习
author: Zhang Ge
date: 2021-08-10 21:47:00 +0800
categories: [实践, 框架学习]
tags: [matplotlib]
---

# 关于本页

matplotlib是一个常用的Python绘图包，本页介绍我在使用过程中的一些操作和经验记录，便于日后参考复用。

![](/assets/img/20211222/matplotlib_sample.png)

<center>图1   matplotlib绘图的关键元素</center>
# 基本使用

## 导入包

```python
import matplotlib.pyplot as plt
```

## A quick and dirty way - 使用plt.xxx

```python
A = [1, 2, 3]
B = [4 ,5, 6]
plt.plot(A, B)
plt.title('test title')
plt.show()
```

以上代码默认建立一张`Figure`，在这张`Figure`上隐式地建立一个`axes`，然后把内容绘制在这个`axes`上。

matplotlib要画出一张图真是有N种方式。而`plt.xxx`尽管方便，但有的功能可能不支持，所以为了方便代码的复用，希望自己稳定地使用一种更加普适的方式。

## 使用ax.xxx

```python
fig, ax = plt.subplots()
print(type(fig))    # <class 'matplotlib.figure.Figure'>
print(type(ax))     # <class 'matplotlib.axes._subplots.AxesSubplot'>
```

`plt.subplots()`的默认值为`ncols=nrows=1`，故以上代码返回的`ax`为单一的一个`AxesSubplot`对象（不可索引），我们可以在这个`ax`上绘图：

```python
ax.plot(A, B)
```

### 绘制多图

```python
n_rows, n_cols = 2, 2
fig, axes = plt.subplots(n_rows, n_cols)
print(type(axes))
print(axes)
```

输出为：

```bash
<class 'numpy.ndarray'>
[[<AxesSubplot:> <AxesSubplot:>]
 [<AxesSubplot:> <AxesSubplot:>]]
```

可以看到返回的`axes`是一个由`AxesSubplot`组成的`numpy.ndarray`对象，我们需要使用常规的索引得到对应的`ax`，然后在其上绘图：

```python
axes[0][0].plot(...)
axes[0][1].plot(...)
axes[1][0].plot(...)
axes[1][1].plot(...)
```

使用`for`循环：

```python
for i in range(n_rows):
    for j in range(n_cols):
        axes[i][j].plot(...)     # plot anything you want on the coresponding ax 
        
plt.show()
```

如果觉得二维的索引写起来有些麻烦，也可以

```python
for i in range(n_rows * n_cols):
    ax = plt.subplot(n_rows, n_cols, i+1)   # because index in plt starts from 1, so we use `i+1` here
    ax.plot(...)

plt.show()
```

### 为图像增加属性

有时需要为图像增加一些属性，如坐标轴`label`，图的`title`等。

```python
ax.set_xlabel('x_label')   # x_label
ax.set_ylabel('y_label')   # y_label
ax.set_title('ax title')   # ax title

fig.suptitle('fig title')  # fig title
```

### 绘制曲线

#### 为曲线增加属性

[marker](https://www.geeksforgeeks.org/matplotlib-markers-module-in-python/)

```python
ax.plot(A, B, marker='o')
```

[color](https://matplotlib.org/stable/gallery/color/named_colors.html)

```python
ax.plot(A, B, color='b')
```

#### 一个`ax`上绘制多条曲线并添加图例

```python
ax.plot(A1, B1, label='plot_1')
ax.plot(A2, B2, label='plot_2')
ax.legend()
```

#### 水平/竖直线

[参考](https://www.delftstack.com/zh/howto/matplotlib/how-to-plot-horizontal-and-vertical-line-in-matplotlib/)

- 水平线：`ax.axhline(y=5, xmin=0.1, xmax=0.9, linestyle="--")`
- 竖直线：`ax.axvline(x=5, ymin=0.1, ymax=0.9, linestyle="--")`

### 绘制直方图
[参考](https://www.geeksforgeeks.org/bar-plot-in-matplotlib/)

# 其他

## 一些操作

- 保存图像：`plt.savefig(PATH)`

- 把图例放在图像外：[参考](https://matplotlib.org/3.5.0/api/_as_gen/matplotlib.pyplot.legend.html)，调节`legend()`参数，一个示例如下：

  ```python
  # plt.figure(figsize=(8, 4.8))   # 可能需要调节画布大小，防止图像本身被图例空间过度压缩
  
  # 把图例的左上角对齐到图的右上角
  plt.legend(loc='upper left', bbox_to_anchor=(1., 1.))
  plt.tight_layout()   # 没有这一行，图例可能被裁切掉
  ```

  

## 图像属性设置

- 设置`Figure`大小：`plt.figure(figsize=(width, height)) ` 
  - 默认是`(6.4, 4.8)` [参考](https://www.geeksforgeeks.org/how-to-change-the-size-of-figures-drawn-with-matplotlib/)

- 设定图像分辨率：`plt.figure(dpi=1200)`

# 参考

- [Medium - What Are the “plt” and “ax” in Matplotlib Exactly?](https://towardsdatascience.com/what-are-the-plt-and-ax-in-matplotlib-exactly-d2cf4bf164a9)
- [matplotlib：先搞明白plt. /ax./ fig再画](https://zhuanlan.zhihu.com/p/93423829)
- [matplotlib gallery](https://matplotlib.org/2.0.2/gallery.html)
- [geeksforgeeks - Matplotlib Tutorial](https://www.geeksforgeeks.org/matplotlib-tutorial/)

