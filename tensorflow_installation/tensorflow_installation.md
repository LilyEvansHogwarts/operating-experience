# Tensorflow Installation

## 1. install Anaconda2-4.3.1-Linux-x86_64.sh

* 由于在官网上下载速度太慢，因此，实际操作中是通过以下命令获取的Anaconda:
 > **wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda2-4.3.1-Linux-x86_64.sh**
* 下载完毕，通过以下命令进行Anaconda安装：
> **bash Anaconda2-4.3.1-Linux-x86_64.sh**
* 测试Anaconda是否安装成功:
![python](python.png)

## 2. 建立名叫tensorflow的计算环境

> ###### Python 2.7
> $ conda create -n tensorflow python=2.7
> ###### Python 3.4
> $ conda create -n tensorflow python=3.4

具体运行结果如下:
![conda create -n tensorflow python=2.7](conda_create.png)

## 3. 激活以及关闭tensorflow的环境

> ###### To activate this environment, use:
> $ source activate tensorflow
> ###### To deactivate this environment, use:
> $ source deactivate tensorflow

具体运行结果如下:
![source activate tensorflow](tensorflow_activate.png)
