+++
title = "GCP小技巧 深度学习配置"
description = "GCP上可利用的算力是庞大的，只要有足够的经费，就可以构建够强的算力"
author = "David Shu"
date = "2021-01-23T20:33:23+08:00"
tags = ["GCP", "deep learning"]
categories = ["GCP"]
removeBlur = true
[[images]]
  src = "https://static.code-david.cn/blog/tensorflow.png"
  alt = "tensorflow"
  stretch = ""
+++

谷歌云上不得不说的一大好处是其具有及其丰富的GPU算力资源，从其开放免费的CoLab就可以看出，我们可以通过合理规划需求，弹性安排所需算力进行深度学习实验，本篇文章以动作识别论文实验复现为例，介绍部署经验。

## 申请资源步骤

- 账号需要进行GPU资源配额申请，google默认是不给GPU配额的
- 创建虚拟机实例，选择具有GPU的区域进行创建
- CPU需要选择N1类型(不然无法使用GPU)，我选择了4核心15G内存
- GPU选择T4，具有15G显存
- 有时会碰到该区域无多余资源可配的情况，这时需要换一个区域配置资源

## 驱动安装
- 以下安装步骤来自于tensorflow官网说明，实测有效，因是google的云服务器，没有网络问题，安装速度很快，下载速度最高我到过0.5GB/s
- 添加环境变量，在`.bashrc`文件添加，并用source生效
```
export PATH=$PATH:/usr/local/cuda/bin
export LD_LIBARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64
```
- 编写一个安装脚本，自动化执行安装，我设置脚本名称为`repo.h·`，脚本内容如下：
```shell
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.2.89-1_amd64.deb
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo dpkg -i cuda-repo-ubuntu1804_10.2.89-1_amd64.deb
sudo apt-get update
wget http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
sudo apt install ./nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
sudo apt-get update
rm -rf repo.sh cuda-repo-ubuntu1804_10.2.89-1_amd64.deb nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
```
- 执行脚本指令，将安装源添加完毕
```shell
sudo sh repo.sh
```
- 执行安装指令
```
# Install NVIDIA driver & development runtime libraries (~4GB)(4min40s)
sudo apt-get install --no-install-recommends \
    cuda-10-0 \
    libcudnn7=7.6.5.32-1+cuda10.0  \
    libcudnn7-dev=7.6.5.32-1+cuda10.0

# Install TensorRT. Requires that libcudnn7 is installed above.
sudo apt-get install -y --no-install-recommends libnvinfer6=6.0.1-1+cuda10.0 \
    libnvinfer-dev=6.0.1-1+cuda10.0\
    libnvinfer-plugin6=6.0.1-1+cuda10.0
```
到这里所有需要安装的驱动均完成安装，整体耗时大约为15min，可见速度还是很快的

## 不同cuda版本安装方法
- 将上述安装指令中对应10-0或10.0的部分进行修改即可安装对应版本，如选装10.1则如下

```
# Install NVIDIA driver & development runtime libraries (~4GB)(4min40s)
sudo apt-get install --no-install-recommends \
    cuda-10-1 \
    libcudnn7=7.6.5.32-1+cuda10.1  \
    libcudnn7-dev=7.6.5.32-1+cuda10.1

# Install TensorRT. Requires that libcudnn7 is installed above.
sudo apt-get install -y --no-install-recommends libnvinfer6=6.0.1-1+cuda10.1 \
    libnvinfer-dev=6.0.1-1+cuda10.1\
    libnvinfer-plugin6=6.0.1-1+cuda10.1
```