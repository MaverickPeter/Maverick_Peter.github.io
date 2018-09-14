---
layout:     post
title:      "Tips for MXNet Learning"
subtitle:   " \"MXNet使用贴士\""
date:       2018-07-20 10:00:00
author:     "xxc"
header-img: "img/post-bg-dl-mxnet.jpg"
catalog: true
tags:
    - Deep Learning

---

- 2017年底已经在 阿里云的 cpu服务器路上配置好了环境，同时在gpu实例上配置了环境并做成了相应的镜像DNN_GPU。
- 学习数据来源：Kaggle
- 学习教程网站：<http://zh.gluon.ai/>

## Windows上配置MXNet GPU版

- **CUDA安装：**

  官网下载对应cuda_xxx.exe，运行exe，精简安装，如果没出错，恭喜。一般会出现VS integration安装失败，需要使用自定义安装，取消vsintegration选项，在安装过程中，到最开始设置的临时文件夹中将CUDAVisualStudioIntegration文件夹拷出，具体步骤见链接：<https://blog.csdn.net/jin739738709/article/details/80819441>

- **安装MXNet CPU版：**

  1. 根据操作系统下载 Miniconda

  2. 下载GLUON教程全部代码的压缩包并解压

  3. 使用 Conda 创建并激活环境。Conda 默认使用国外站点来下载软件，以下可选项配置使用国内镜像加速下载:

     ```
     # 使用清华 conda 镜像。
     conda config --prepend channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
     
     # 或者选用科大 conda 镜像。
     conda config --prepend channels http://mirrors.ustc.edu.cn/anaconda/pkgs/free/
     ```

  4. 使用 conda 创建虚拟环境并安装本书需要的软件

     ```
     conda env create -f environment.yml
     ```

  5. 激活之前创建的环境

     ```
     activate gluon
     ```

  6. 打开 Juputer 笔记本

     ```
     jupyter notebook
     ```
- **安装MXNet GPU版：**

  1. 如果先前装有CPU版，需要先卸载CPU版本 MXNet。如果没有安装虚拟环境，可以跳过此步。否则假设你已经完成了安装，那么先激活运行环境，然后卸载 CPU 版本的 MXNet：pip uninstall mxnet
  2. 退出虚拟环境（gluon）使用deactivate， 更新依赖为 GPU 版本的 MXNet。使用文本编辑器打开之前文件夹下的文件environment.yml，将里面的“mxnet”替换成对应的 GPU 版本。如mxnet-cu80，保存后退出。
  3. conda env update -f environment.yml（如果没有安装过gluon环境，则为conda env create -f environment.yml）

## Linux上配置MXNet GPU版

根据操作系统下载 Miniconda

```
sh Miniconda3-latest-Linux-x86_64.sh
```

安装时会显示使用条款，按“↓”继续阅读，按“Q”退出阅读。之后需要回答下面几个问题：

```
Do you accept the license terms? [yes|no]
[no] >>> yes
Do you wish the installer to prepend the Miniconda3 install location
to PATH in your /home/your_name/.conda ? [yes|no]
[no] >>> yes
```

之后和在windows上一致。激活之前创建的环境时代码为：

```
source activate gluon
```