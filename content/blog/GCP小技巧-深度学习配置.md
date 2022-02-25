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

谷歌云上不得不说的一大好处是其具有及其丰富的GPU算力资源，从其开放免费的CoLab就可以看出，我们可以通过合理规划需求，弹性安排所需算力进行深度学习实验，本篇文章以动作识别论文实验为例，介绍部署方法。同时介绍在服务器上利用rclone挂载google drive的方法，该方法可快速load在google drive上的数据集等资源

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
- 编写一个安装脚本，自动化执行安装，我设置脚本名称为`repo.sh`，脚本内容如下：
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
sh repo.sh
```
- 执行安装指令
```
# Install NVIDIA driver & development runtime libraries (~4GB)(4min40s)
sudo apt install cuda-drivers-470
sudo apt install cuda-toolkit-11-4
sudo apt-get install --no-install-recommends \
    libcudnn8=8.2.4.15-1+cuda11.4 \
    libcudnn8-dev=8.2.4.15-1+cuda11.4

# Install TensorRT. Requires that libcudnn7 is installed above.
sudo apt-get install -y --no-install-recommends libnvinfer8=8.2.3-1+cuda11.4 \
    libnvinfer-dev=8.2.3-1+cuda11.4\
    libnvinfer-plugin8=8.2.3-1+cuda11.4
```
到这里所有需要安装的驱动均完成安装，整体耗时大约为15min，可见速度还是很快的

## 动作识别实验部署
- 推荐安装anaconda，这里不要使用sudo
```shell
wget https://repo.anaconda.com/archive/Anaconda3-2020.07-Linux-x86_64.sh
sh Anaconda3-2020.07-Linux-x86_64.sh
rm -rf Anaconda3-2020.07-Linux-x86_64.sh
```
- 以[mmskeleton](https://github.com/open-mmlab/mmskeleton)为例，由于服务器在境外，conda源和pip源也不用修改
```shell
# Create a conda virtual environment and activate it:
source .bashrc
conda create -n open-mmlab python=3.7 -y
conda activate open-mmlab

# Install PyTorch and torchvision (CUDA is required):
conda install pytorch==1.2.0 torchvision==0.4.0 cudatoolkit=10.0 -c pytorch -y

# Clone mmskeleton from github:
git clone https://github.com/open-mmlab/mmskeleton.git

# Install requirements
cd mmskeleton
pip install -r requirements.txt
pip install mmcv==0.4.3
conda install cython

# Install mmskeleton:
python setup.py develop

# Install nms for person estimation:
cd mmskeleton/ops/nms/
python setup_linux.py develop
cd ../../../

# Install mmdetection for person detection:
python setup.py develop --mmdet

# To verify that mmskeleton and mmdetection installed correctly, use:
python mmskl.py pose_demo
 
```

经过以上步骤就已经部署完毕了，整体步骤也不算多，主要分Nvidia驱动安装和实验部署两部分。

## 使用google drive的资源

数据集、模型参数之类的资源很多人是使用google drive进行分享，那么在服务器上如何快速load这类资源呢，方法是使用[rclone](https://rclone.org)程序，Rclone是一个命令行程序，用于管理云存储上的文件，超过40种云存储产品支持rclone，当然也包括了google drive。

### 安装方式
官方已经提供了非常方便的安装脚本，具体可以查看官网文档，指令如下：
```shell
curl https://rclone.org/install.sh | sudo bash
```
根据提示运行rclone config进行配置，以下是配置方法:
```shell
rclone config

# 第一个选项选择
n) New remote

# 名字自定义，我填写remote
name> remote

# 选择13号的google drive，填写13
13 / Google Drive
   \ "drive"
Storage> 13

# 使用默认方式会损失一定性能，若要追求高性能则需去看官网文档配置google api
# 这里我使用默认方式，将以下两个选项均留空即可
client_id>
client_secret>

# 配置使用权限，可以是只读，可以是只管理rclone相关文件，也可以给全部权限
# 这里我选择全部，填写1
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
scope> 1

# 选择rclone的根目录挂载点，默认是直接挂载到google drive根目录
# 我这里选择默认，即留空
root_folder_id>

# 选择以SA方式验证的文件，也可以直接使用交互方式登录
# 不选择则默认以交互式方式登录，这里留空默认
service_account_file>

# 在远程主机上不选择auto config，填写n
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes (default)
n) No
y/n> n

# 复制得到的一段网址进行验证得到验证码
# 将验证码填写到下方即可
Enter verification code>

# 不选择team drive，填写n
Configure this as a team drive?
y) Yes
n) No (default)
y/n> n

# 最后确认以下token，确定配置完成
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y

# 配置已经完成，直接退出就好了，然后就可以进行google drive操作了
Current remotes:

Name                 Type
====                 ====
remote               drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

### 使用方法
以上为配置方式，接下来介绍最简单的使用方法，仅仅拷贝数据仅需要了解以下三条指令：
```shell
# 列出对应路径下的所有目录
rclone lsd remote:/
          -1 2020-07-14 07:23:53        -1 colab
          -1 2021-01-24 02:11:26        -1 data_set
          -1 2021-01-04 10:25:09        -1 音乐

# 列出对应路径下的所有文件
rclone ls remote:/
8635721718 st-gcn-processed-data.zip
    84517 colab/linear_regression.ipynb

# copy文件，如要拷贝某个文件到本地，注意路径的尾部/不能丢，-P参数看进度
# rclone copy <google drive中文件路径> <本地路径> -P
rclone copy remote:/st-gcn-processed-data.zip /home/sdw/ -P
```
传输速度也比较可观：

![rclone speed](https://static.code-david.cn/blog/zpUxrB.png)

使用起来也是非常容易，更多高阶操作查看这里[rlcone 指令](https://rclone.org/commands/)

### rclone还解决了什么问题
目前在服务器端，若是直接使用别人分享的google下载公开链接，使用wget或curl会有或多或少的问题，此时可以通过将链接创建到自己的google drive快捷方式，然后使用rclone进行下载即可，且由于使用Google骨干网，下载速度也是很不错的，速度瓶颈甚至是本地硬盘的读写速度。

![分享链接google drive](https://static.code-david.cn/blog/1UUtOu.png)

## 实验结果复现
根据以上手段部署环境，将数据集load到指定路径即可根据论文提供的操作方式进行实验复现了，具体步骤可以参考各自论文提供的方法，本篇文章就不再赘述了