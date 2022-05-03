---
title: Ubuntu下nvidia driver和cuda版本管理
author: Zhang Ge
date: 2020-12-05 18:15:00 +0800
categories: [实践, 环境配置]
tags: [ubuntu, nvidia driver, cuda]
---

# 动机

记录下在`Ubuntu 18.04`下`nvidia driver 455.45`与`cuda 10.1`的安装过程。备后续重复安装时直接参照。

# 版本对应关系
## cuda与nvidia driver![https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html](https://img-blog.csdnimg.cn/20201205174620889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU2MTA2NQ==,size_16,color_FFFFFF,t_70)
可以看到，Driver对于CDUA版本是向下兼容的。表格内容可能过时，可以点击[nvidia官网链接](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)查看最新更新。

## cuda与pytorch
较老版本的Pytorch：参见[本页](https://pytorch.org/get-started/previous-versions/)

最新版Pytorch：[Pytorch首页](https://pytorch.org/)给出了最新版Pytorch可用的组合和对应的安装命令。

以下以在Ubuntu 18.04下，安装nvidia driver 455.45与cuda 10.1 为例，记录安装过程。

# 安装/更新英伟达驱动(nvidia driver)
## 卸载原驱动(如若需要)
```bash
sudo apt-get purge "nvidia*"

# ./NVIDIA-Linux-x86_64-390.48.run --uninstall
```
## 下载正确版本的驱动
### 查看显卡型号
 ```bash
 lspci | grep -i nvidia
 ```
 > 部分老版本Ubuntu(如16.04)输出为一十六进制数，可以在[The PCI ID Repository](http://pci-ids.ucw.cz/mods/PC/10de?action=help?help=pci)`See also`一栏下输入框查询该代码对应的显卡型号。




### 下载相应的驱动版本
在[nvidia驱动下载页](https://www.nvidia.cn/geforce/drivers/)手动搜索并下载所需的驱动版本。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201205164600920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDU2MTA2NQ==,size_16,color_FFFFFF,t_70)

## 安装新驱动
#### 预备
首先，禁用系统自带的`nouveau`驱动
```bash
#创建配置文件
sudo vim /etc/modprobe.d/blacklist-nouveau.conf 
 
# 在打开的配置文件里添加如下内容
blacklist nouveau
options nouveau modeset=0
 
# 更新
sudo update-initramfs -u
```
> Tips：打开vim后按`inset`开始编辑，编辑结束后vim的退出操作为：先按`Esc`，然后输入`:wq`

**重启**。（重启后分辨率可能会暂时性下降。在安装NVIDIA驱动完成后，重启即可恢复）

重启后在终端输入

```bash
lsmod | grep nouveau
```
如果输出为空，则禁用成功。


### 开始安装
进入刚才下载的驱动所在的目录，执行以下命令进行安装
```bash
sudo sh NVIDIA-Linux-x86_64-<your-driver-version>.run
```
按照默认设置，一路回车，不出意外便可顺利结束安装。



### 测试
```bash
nvidia-smi
```
输出
```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 455.45.01    Driver Version: 455.45.01    CUDA Version: 11.1     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  GeForce GTX TIT...  Off  | 00000000:01:00.0  On |                  N/A |
| 22%   58C    P8    21W / 250W |     90MiB / 12211MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
```
### 实际安装过程遇到过的一些问题
#### Failed CC version check. Bailing out!
```bash
Compiler version check failed:
The major and minor number of the compiler used to
compile the kernel:
gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.4) 
does not match the compiler used here:
cc (Ubuntu 4.9.4-2ubuntu1~14.04.1) 4.9.4
```
将本机的`gcc`默认版本切换为当前编译所需版本即可(此处为将`4.9.4`切换为`4.8.4`)。

[参考1-ubuntu下多个gcc版本切换](https://blog.csdn.net/astrotycoon/article/details/8069621) 

[参考2-Ubuntu 16.04 GCC 7 & G++ 7 安装](https://blog.csdn.net/calvinpaean/article/details/90691315)。

> - 查看当前使用的`gcc`版本：`gcc -v`
>   - 查看本机已安装哪些`gcc`版本：`ls /usr/bin/gcc*`
> - 切换`gcc`版本：`sudo update-alternatives --config gcc`
> 	- 根据输出，输入希望切换至的`gcc`版本对应的编号，即可完成切换。

#### no cc
```bash
sudo apt update
sudo apt install build-essential
```

#### 对ubuntu系统自动更新的尝试
[这种方法](https://blog.csdn.net/u010420283/article/details/104020531)很是简便，但在我的电脑上尝试时，虽然可以获取相应更新，但是重启后驱动无法连接，也无法正常加载桌面。但也记录一下，没有行通可能只是我电脑本身的原因。

首先，更新系统软件源信息
```bash
sudo add-apt-repository ppa:graphics-drivers/ppa && sudo apt update
```
然后来到设置-软件和更新-附加驱动，选择希望更新至的驱动版本，点击应用更改，完成后重启。

# 安装/切换cuda版本
[参考链接](https://blog.csdn.net/qq_45057749/article/details/107320354)
## 下载并安装cuda
从[CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive)找到所需的cuda版本，选择`runfile`类型文件下载。

进入刚才下载文件所在的目录，执行以下命令进行安装（执行本命令前请先看下面的**注**）
```bash
sudo ./cuda_<your_version>_linux.run  
```
希望一切顺利！

**注：**如果已经安装了nvidia驱动（常发生在安装新版本cuda的情况），这一步不需要再安装，取消默认选中的NVIDIA Driver（不取消不仅可能会安装失败，还可能导致重启时循环登录），如下

```bash
CUDA Installer                                                                 │
│ - [ ] Driver                                                                 │
│      [ ] 450.51.06                                                           │
│ + [X] CUDA Toolkit 11.0                                                      │
│   [ ] CUDA Samples 11.0                                                      │
│   [ ] CUDA Demo Suite 11.0                                                   │
│   [ ] CUDA Documentation 11.0                                                │
│   Options                                                                    │
│   Install      
```

按回车取消其他选项，仅保留`CUDA Toolkit 11.0`，选择`Install`继续安装。

## cuda环境设置

在`~/.bashrc`末尾添加
```bash
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
```
> 上面的路径都是指向`/usr/local/cuda`的软连接，而不是指向某一个具体的`cuda`版本，这样方便于后面进行`cuda`版本的切换。

执行以下命令使改动立即生效
```bash
source ~/.bashrc
```
叮叮，安装完成。

## 切换cuda版本
查看当前使用的cuda版本：`nvcc -V `

当电脑里安装了多个cuda版本，而想从一个版本切换到另外一个版本时，可以将软链接重指向至新版本：
```bash
cd /usr/local/ # 切到cuda所在目录下
sudo rm -rf ./cuda  # 删去原先使用的cuda版本的软链接
sudo ln -s cuda-10.1 cuda  # 创建一个新的软链接到希望切换到的cuda版本
```
再使用`nvcc -V `看看`cuda`使用版本是否已经发生改变叭。

## 卸载cuda

```bash
To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-11.1/bin
```

# 使用过程中出现过的问题及解决方案
- [NVIDIA-SMI has failed because it couldn’t communicate with the NVIDIA driver](https://wangpei.ink/2019/01/19/NVIDIA-SMI-has-failed-because-it-couldn't-communicate-with-the-NVIDIA-driver的解决方法/)

- 安装cuda后出现图形化界面循环登录的现象。

  - [参考](https://www.cnblogs.com/dinghongkai/p/11268976.html)

  - 这是因为运行cuda安装文件时，没有取消安装NVIDIA驱动选项导致的。

  - 解决：

    - 卸载安装的CUDA

      ```bash
      sudo /usr/local/cuda-10.1/bin/cuda-uninstaller
      ```
	    
	  - 重新安装CUDA（注意取消安装NVIDIA驱动的选项）



# 其他

显示Nvidia显卡型号（已安装Nvidia驱动后）：`nvidia-smi -L`