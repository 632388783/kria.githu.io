---
title: 关于 x86 架构服务器下载 arm64 docker 容器的说明
index_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
banner_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
cover: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
date: 2025-03-07 16:37:31
tags: [docker]
categories: [docker]
toc: true
permalink: /关于x86架构服务器下载arm64 docker容器的说明/
description: 关于 x86 架构服务器下载 arm64 docker 容器的说明
comment: 'valine'
---
# 关于 x86 架构服务器下载 arm64 docker 容器的说明

## 一、在 x86 服务器上配置 docker 镜像源

在 x86 架构服务器开展 arm64 docker 容器下载工作前，首要任务是合理配置 docker 镜像源。不同操作系统配置方式略有差异，以常见的 Ubuntu 系统为例，用户需编辑`/etc/docker/daemon.json`文件。若该文件不存在，则需手动创建。在文件中添加如下内容（此处以配置国内常用的阿里云镜像源为例）：



```json
{
  "registry-mirrors": ["https://<your-aliyun-mirror-id>.mirror.aliyuncs.com"]
}
```

`<your-aliyun-mirror-id>`需替换为用户在阿里云容器镜像服务中申请的专属镜像加速地址。完成文件编辑保存后，执行`sudo systemctl restart docker`命令，使配置生效。配置完成后，需仔细检查服务器网络连接状态，可通过`ping`命令测试与外部网络的连通性，如`ping ``www.baidu.com`，确保服务器能正常联网，为后续顺利下载 docker 镜像奠定基础。

## 二、下载 arm64 docker 镜像

完成镜像源配置且网络正常后，即可执行 arm64 docker 镜像下载命令：



```shell
docker pull contain_name --platform linux/arm64
```

此命令中，`contain_name`是用户需要下载的目标容器名称，务必准确填写，否则将无法获取正确镜像。例如，若要下载`ubuntu:latest`的 arm64 版本镜像，`contain_name`就应填写为`ubuntu:latest`。在执行下载命令过程中，网络稳定性至关重要。若网络波动较大，可能导致下载中断，此时可尝试重新执行下载命令。同时，需关注服务器磁盘空间，确保有足够容量存储下载的镜像文件。可通过`df -h`命令查看磁盘使用情况。

## 三、保存镜像为 tar 文件

当 arm64 docker 镜像下载完成后，为便于在不同服务器间传输，需将镜像以 tar 文件格式保存。执行如下命令：



```shell
docker save contain_name:version -o contain_name.tar
```

这里的`contain_name`为容器名称，`version`代表镜像版本号。比如对于`ubuntu:20.04`镜像，`contain_name`是`ubuntu`，`version`是`20.04`。执行该命令后，系统会将指定版本的镜像打包成`contain_name.tar`文件。在保存过程中，要留意文件保存路径。若未指定绝对路径，文件将保存在当前执行命令的目录下。可通过`ls -l`命令确认文件是否成功生成及所在位置。此外，由于镜像文件通常较大，保存过程可能需要一定时间，期间需保持系统资源稳定，避免因资源不足导致保存失败。

## 四、上传镜像至 arm64 服务器

将保存好的`contain_name.tar`镜像文件传输至目标 arm64 服务器时，选用 TCP 协议较为可靠。常用的传输工具如`scp`（基于 SSH 协议，而 SSH 底层使用 TCP 协议）。假设 x86 服务器的 IP 地址为`192.168.1.100`，arm64 服务器的 IP 地址为`192.168.1.101`，且 x86 服务器上`contain_name.tar`文件位于`/home/user/`目录，在 x86 服务器上执行如下命令上传文件：



```shell
scp /home/user/contain_name.tar user@192.168.1.101:/home/user/
```

其中，`user`需替换为 arm64 服务器的用户名。执行命令后，系统会提示输入 arm64 服务器的用户密码，输入正确密码后即可开始传输。传输过程中，可通过观察命令行进度条了解传输进度。若传输中断，可根据错误提示排查问题，如网络中断、权限不足等，并重新尝试传输。

## 五、在 arm64 服务器上加载并运行镜像

镜像文件成功上传至 arm64 服务器后，需执行加载操作，具体命令如下：



```shell
docker load -i /path/contain_name.tar
```

`/path/`需替换为实际存放`contain_name.tar`文件的路径，例如若文件存放在`/home/user/`目录，则路径填写为`/home/user/`。加载完成后，可通过`docker images`命令查看已加载的镜像列表，确认镜像是否成功加载。确认无误后，即可按照常规`docker run`命令运行镜像。运行命令时，可根据实际需求添加各种参数，如指定容器运行时使用的端口映射、挂载数据卷等。例如，运行一个基于`ubuntu:latest`镜像的容器，并将容器的 80 端口映射到服务器的 8080 端口，同时挂载`/host/data`目录到容器的`/container/data`目录，命令如下：



```shell
docker run -p 8080:80 -v /host/data:/container/data ubuntu:latest
```

在运行过程中，可通过`docker logs`命令查看容器运行日志，以便及时排查问题。