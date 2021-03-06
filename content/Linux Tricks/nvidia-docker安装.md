---
title: "Nvidia-Docker安装"
layout: page
date: 2018-06-02 00:00
---
[TOC]

# 写在前面
nvidia-docker是一个可以使用GPU的docker，nvidia-docker是在docker上做了一层封装，通过nvidia-docker-plugin，然后调用到docker上，其最终实现的还是在docker的启动命令上携带一些必要的参数。

# 安装docker-ce和nvidia驱动
在安装nvidia-docker之前，还是需要安装docker的。并且, 官网上介绍本地机器上**必须安装NVIDIA driver和Docker19.03以上的版本**, 如果docker版本过低, 请重新安装或升级Docker. [Docker-ce安装教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

# 安装nvidia-docker
安装过程主要参考[官网安装](https://github.com/NVIDIA/nvidia-docker), 安装nvidia-docker之前必须确保已经安装了NVIDIA driver, 且安装的Docker版本>19.03.

如果是第一次使用GPUs和Docker的用户, 可以直接使用下面的命令:
```
# Add the package repositories
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
$ curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

$ sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
$ sudo systemctl restart docker
```
运行过程如下:
<center><img src="/wiki/static/images/docker/nvidia-docker_install.png" alt="nvidia-docker"/></center>
nvidia-docker安装完成之后, 重启dokcer.

## docker中使用
方法一
```
docker run -it \
-p 8080:8080 \
-p 8081:8081 \
-v /home/gpyz/workshop:/home/li/workshop \
-v /usr/include:/usr/include \
--privileged=True \
--gpus all \
[REPOSITORY:TAG] /bin/bash
```
方法二
```
nvidia-docker run -it \
-p 8080:8080 \
-p 8081:8081 \
-v /home/gpyz/workshop:/home/li/workshop \
-v /usr/include:/usr/include \
--privileged=True \
--runtime=nvidia \
[REPOSITORY:TAG] /bin/bash

```

方法一
```bash
docker run -it \
-p 8987:8987 \
-v /home/gpyz/workshop:/home/work/workshop \
-v /home/gpyz/soft/ML:/home/work/soft \
--restart always \
--privileged=True \
--gpus all \
--name quant_flask \
sthsf/ubuntu_base:0.09 /bin/bash
```

```bash
docker run -it \
-p 8022:22 \
--restart always \
--privileged=True \
--runtime=nvidia \
--name quant_ \
sthsf/ubuntu_base:0.1 /usr/sbin/sshd -D
```

方法二
```bash
nvidia-docker run -it \
-p 8022:22 \
-p 8084:8084 \
-v /home/gpyz/workshop:/home/li/workshop \
-v /home/gpyz/soft/ML:/home/li/soft/ML \
--restart always \
--privileged=True \
--runtime=nvidia \
--name ubuntu3 \
mdjaere/tensorflow-notebook-gpu:latest /bin/bash
```



PYTHONIOENCODING=utf-8 python run.py

# 参考文献
[如何在ubuntu上安装nvidia-docker同时与宿主共享GPU cuda加速](https://www.liangzl.com/get-article-detail-3784.html)

[ubuntu16.04下docker和nvidia-docker安装](https://blog.csdn.net/qq_41493990/article/details/81624419)

[安装NVIDIA-DOCKER](https://www.jianshu.com/p/f25ccedb996e)

[nvidia-docker](https://github.com/NVIDIA/nvidia-docker)

[ubuntu16.04下docker和nvidia-docker安装](https://blog.csdn.net/qq_41493990/article/details/81624419)

[NCCL](https://github.com/NVIDIA/nccl)

[Installation (Native GPU Support)](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(Native-GPU-Support))

[Centos7使用Docker快速搭建深度学习环境](https://zhongneng.github.io/2018/12/04/docker-env/)

[使用docker在Ubuntu上安装TensorFlow-GPU](https://blog.csdn.net/qq_29300341/article/details/84754970)