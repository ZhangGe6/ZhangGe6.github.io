---
title: Pytorch Quantization
author: Zhang Ge
date: 2021-03-23 16:27:00 +0800
categories: [实践, 框架学习]
tags: [pytorch, quantization]
math: true
---

Pytorch目前(1.7)只支持在CPU上量化，从fp32量化到int8.

[Pytorch Quantization](https://pytorch.org/docs/stable/quantization.html)

# supported operations

[QUANTIZATION OPERATION COVERAGE](https://pytorch.org/docs/stable/quantization-support.html)

[torch.quantization functions](https://pytorch.org/docs/stable/torch.quantization.html)

# Dynamic quantization

[pytorch官方](https://pytorch.org/tutorials/recipes/recipes/dynamic_quantization.html#dynamic-quantization)

使用起来很简单，只需要

```python
import torch.quantization

quantized_model = torch.quantization.quantize_dynamic(
    model,   # the original model
    {torch.nn.Linear},   # a set of layers to dynamically quantize
    dtype=torch.qint8    # the target dtype for quantized weights
)

out = quantized_model(input)
```

Dynamic quantization实际使用起来最为简单，但是一般效果也是最差的。

# Static quantization

[用法](https://pytorch.org/docs/stable/quantization.html)  => static quantization API Example

```python
# main.py

# model must be set to eval mode for static quantization logic to work
model.eval()
# Use 'fbgemm' for server inference and 'qnnpack' for mobile inference.
model.qconfig = torch.quantization.get_default_qconfig('fbgemm')

# Fuse the activations to preceding layers
model_fused = copy.deepcopy(model)
model_fused.fuse_modules()

print(model_fused)

# inserts observers in the model that will observe activation tensors during calibration
model_prepared = torch.quantization.prepare(model_fused)
print(model_prepared)

# calibrate the prepared model to determine quantization parameters for activations using a representative dataset
# print("start calibrating...")
for data, target in tqdm(testloader):
	output = model_prepared(data)
print("calibrating done!")

# Convert the observed model to a quantized model.
model_int8 = torch.quantization.convert(model_prepared)

out = model_int8(input)
```

```python
# model.py
class M(torch.nn.Module):
    def __init__(self):
        super(M, self).__init__()
        self.conv = ...
        # QuantStub converts tensors from floating point to quantized
        self.quant = torch.quantization.QuantStub()
        # DeQuantStub converts tensors from quantized to floating point
        self.dequant = torch.quantization.DeQuantStub()

    def forward(self, x):
        # manually specify where tensors will be converted from floating
        # point to quantized in the quantized model
        x = self.quant(x)
        x = self.conv(x)
        ...
        # manually specify where tensors will be converted from quantized
        # to floating point in the quantized model
        x = self.dequant(x)
        return x
    
    # an fuse_modules example
    def fuse_modules(self):
        torch.quantization.fuse_modules(self, [["conv1", "bn1", "relu"]], inplace=True)
        for module_name, module in self.named_children():
            if "layer" in module_name:
                for basic_block_name, basic_block in module.named_children():
                    torch.quantization.fuse_modules(basic_block, [["conv1", "bn1", "relu1"], ["conv2", "bn2", "relu2"], ["conv3", "bn3"]], inplace=True)
                    for sub_block_name, sub_block in basic_block.named_children():
                        if sub_block_name == "downsample":
                            torch.quantization.fuse_modules(sub_block, [["0", "1"]], inplace=True)
```



# Quantization-aware training

没来得及看。

# 大小坑

## +操作

参考：[Pytorch discuss](https://discuss.pytorch.org/t/supported-quantized-tensor-operations/71688/5)， [Pytorch source code](https://github.com/pytorch/pytorch/blob/master/torch/nn/quantized/modules/functional_modules.py#L42)

对于量化后的数，使用

```python
quant_functional = nn.quantized.FloatFunctional()
c = quant_functional.add(a, b)   # where a and b are both quantized tensor
```

替代

```
c = a + b
```

## 不做fuse_modules可能会带来错误

[参考](https://leimao.github.io/blog/PyTorch-Static-Quantization/)

Sometimes, layer fusion is compulsory, since there are no quantized layer implementations corresponding to some floating point layers, such as `BatchNorm`.

## ReLU不要复用

[参考](https://leimao.github.io/blog/PyTorch-Static-Quantization/)

For example, in ordinary FP32 model, we could define one parameter-free `relu = torch.nn.ReLU()` and reuse this `relu` module everywhere. However, if we want to fuse some specific `ReLU`s, the `ReLU` modules have to be explicitly separated. So in this case, we will have to define `relu1 = torch.nn.ReLU()`, `relu2 = torch.nn.ReLU()`, etc.

## 多输入/多输出下的QuantStub和DeQuantStub设置

**Q：当网络输入有多个分支，比如FPN，需要设置多个QuantStub吗？如果需要，要怎么设定？如果有多个输出，需要设置多个DeQuantStub吗？**

**A：**

> 注：以下内容均基于个人理解（和相关实验），仅供参考。

理论上由于每个quantStub(之后会convert成Quantize类)都会对应自己的scale和zeropoint等参数，对于不同的分支是需要设定不同的QsuantStub的。但要注意一下具体的写法。

下面这个写法是不对的：

```python
def __init__(self, ...)
    self.quant0 = torch.quantization.QuantStub()
    self.quant1 = torch.quantization.QuantStub()
    self.quant_list = [self.quant0, self.quant1]
    
def forward(self, inputs):
    for i, input in enumerate(inputs):
        input = self.quant_list[i](input)
        # ============== # 
        print(self.quant0)               
        # out: Quantize(scale=tensor([1.]), zero_point=tensor([0]), dtype=torch.quint8)
        print(self.quant_list[0])        
        # out: QuantStub((activation_post_process): HistogramObserver())
        # ============== #
```

为什么呢？如果在`forward`函数中如上输出`self.quant0`和`self.quant_list[0]`，会发现它们已经不是一个东西了。这是由于`self.quant0`作为一个`QuantStub()`类型，在forward时，被替换为了`Quantize`类（`torch/quantization/quantize.py/_convert()`），而`quant_list`中的元素却（因为一些我现在还解释不清的原因）仅仅是加了一个`Observer`。

一个可行的办法是这样写：

```python
def __init__(self, ...)
    self.quant0 = torch.quantization.QuantStub()
    self.quant1 = torch.quantization.QuantStub()
    
def forward(self, inputs):
    for i, input in enumerate(inputs):
        quant = getattr(self, "quant" + str(i))
        input = quant(input)
```

如果有多个输出，应该是不需要设置多个`DeQuantStub`的，因为它所`convert`到的`DeQuantize`是一个parameter free的类型。（其定义如下）

```python
# /torch/nn/quantized/modules/__init__.py

class DeQuantize(torch.nn.Module):
    def __init__(self):
        super(DeQuantize, self).__init__()

    def forward(self, Xq):
        return Xq.dequantize()

    @staticmethod
    def from_float(mod):
        return DeQuantize()
```

## RuntimeError: Trying to create tensor with negative dimension -4398046511104: [-4398046511104]
[参考](https://discuss.pytorch.org/t/combine-histograms-histogram-with-output-range-torch-zeros-nbins-downsample-rate-device-orig-hist-device-runtimeerror-trying-to-create-tensor-with-negative-dimension-4398046511104-4398046511104/89086)

The initial error was due to **the histogram observer getting a tensor with same values or all zero values**. CHECK IT!

# 一次自定义Pytorch量化算子(PReLU)的收获记录

:star2:[cpp doc index](https://pytorch.org/cppdocs/genindex.html)

[PyTorch C++ API class-hierarchy](https://pytorch.org/cppdocs/api/library_root.html#class-hierarchy)

## [PYTORCH C++ API](https://pytorch.org/cppdocs/index.html#pytorch-c-api)

These pages provide the documentation for the public portions of the PyTorch C++ API. This API can roughly be divided into five parts:

- **ATen**: The foundational tensor and mathematical operation library on which all else is built.
- **Autograd**: Augments ATen with automatic differentiation.
- **C++ Frontend**: High level constructs for training and evaluation of machine learning models.
- **TorchScript**: An interface to the TorchScript JIT compiler and interpreter.
- **C++ Extensions**: A means of extending the Python API with custom C++ and CUDA routines.

### ATen

[namespace AT](https://pytorch.org/cppdocs/api/namespace_at.html)

at::Tensor和torch::Tensor的区别

> The `at::Tensor` class in ATen is not differentiable by default. To add the differentiability of tensors the autograd API provides, you must use tensor factory functions from the torch:: namespace instead of the at:: namespace. For example, while a tensor created with at::ones will not be differentiable, a tensor created with torch::ones will be.

### Autograd

```cpp
#include <torch/csrc/autograd/variable.h>
#include <torch/csrc/autograd/function.h>

torch::Tensor a = torch::ones({2, 2}, torch::requires_grad());
torch::Tensor b = torch::randn({2, 2});
auto c = a + b;
c.backward(); // a.grad() will now hold the gradient of c w.r.t. a.
```

### C++ Frontend

[THE C++ FRONTEND of Pytorch](https://pytorch.org/cppdocs/frontend.html)

Pytorch并非就是用Python来写或者只能用Python，Python是它的一个前端，是它和用户交互的一种方式。它还有C++的前端。
>
> A word of warning: Python is not necessarily slower than C++! The Python frontend calls into C++ for almost anything computationally expensive (especially any kind of numeric operation), and these operations will take up the bulk of time spent in a program. If you would prefer to write Python, and can afford to write Python, we recommend using the Python interface to PyTorch. However, if you would prefer to write C++, or need to write C++ (because of multithreading, latency or deployment requirements), the C++ frontend to PyTorch provides an API that is approximately as convenient, flexible, friendly and intuitive as its Python counterpart. The two frontends serve different use cases, work hand in hand, and neither is meant to unconditionally replace the other.

### TorchScript

[See here](https://pytorch.org/cppdocs/index.html#torchscript)

[LOADING A TORCHSCRIPT MODEL IN C++](https://pytorch.org/tutorials/advanced/cpp_export.html)

### C++ Extensions

*C++ Extensions* offer a simple yet powerful way of accessing **all of the above interfaces** for the purpose of extending regular Python use-cases of PyTorch.

[一个来源(暂)不明的网站](https://www.ccoderun.ca/programming/doxygen/pytorch/namespaceat.html)

- 找到了全网很难找的[`at::parallel_for`](https://www.ccoderun.ca/programming/doxygen/pytorch/namespaceat.html#af9f402d7954ef765d351879c6d17fe74)的定义（实际上发现在[Pytorch源码](https://github.com/pytorch/pytorch)目录下查函数定义是最清楚的）

## 其他

**关于at::parallel_for的解释**

```cpp
// pytorch/aten/src/ATen/native/Activation.cpp#L434

 at::parallel_for(0, input_numel, 1000, [&](int64_t start, int64_t end) {
    for (auto i = start; i < end; i++) {
      scalar_t input_data_val = input_data[i];
      // to allow for compiler optimization, here splitting into two lines:
      scalar_t r = (input_data_val > 0) ? scalar_t(1) : weight_val;
      result_data[i] = r * input_data_val;
    }
  });
```

这里用到了C++的lambda表达式，可参考[这里](https://www.cnblogs.com/DswCnblog/p/5629165.html)

来到Pytorch源码，看看`at::parallel_for`的定义

```cpp
// pytorch/aten/src/ATen/ParallelNative.h

template <class F>
inline void parallel_for(
    const int64_t begin,
    const int64_t end,
    const int64_t grain_size,
    const F& f) {
  TORCH_CHECK(grain_size >= 0);
  if (begin >= end) {
    return;
  }
  if ((end - begin) < grain_size || in_parallel_region()) {
    f(begin, end);
    return;
  }
  internal::_parallel_run(
      begin,
      end,
      grain_size,
      [f](int64_t start, int64_t end, size_t /* unused */) {
        f(start, end);
      }
  );
}
```

可以看到，f是会以输入的begin和end作为参数的。开头段代码也就好理解了。

# 参考

:star: [A developer-friendly guide to model quantization with PyTorch](https://spell.ml/blog/pytorch-quantization-X8e7wBAAACIAHPhT)



[知乎-PyTorch的量化](https://zhuanlan.zhihu.com/p/299108528)



[PyTorch Static Quantization - LEI MAO'S LOG BOOK](https://leimao.github.io/)

