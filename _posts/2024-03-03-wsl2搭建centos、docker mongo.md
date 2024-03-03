---
layout: post
title: wsl2搭建centos、docker mongo
key: 202403030910
tags: docker wsl
---

# wsl安装centos
wsl推出的早期，官方并没有提供centos镜像，民间方案大多是使用[CentWSL](https://github.com/wsldl-pg/CentWSL)。wsl2推出后，官方有了centos方案[Import any Linux distribution to use with WSL](https://learn.microsoft.com/en-us/windows/wsl/use-custom-distro)，安装过程需要一些动手能力，安装说明非常详尽，本文不用过多复述。PS：截至到2024.3.1，centos并未上架`microsoft store`。

以下几点需要非常注意：
1. 确保你的window系统版本支持wsl2 [WSL enabled with a Linux distribution installed running WSL 2](https://learn.microsoft.com/en-us/windows/wsl/install-manual)，从未安装参考官方文档[How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
2. 确保导入后的centos，是在wsl2版本中工作，通过`wsl -l -v`检查各个系统使用的wsl版本
3. 手动安装后，确保[开启systemd](https://learn.microsoft.com/en-us/windows/wsl/wsl-config#systemd-support)，否则syctemctl在centos里提示不可用

以下是我实验的系统环境
```bat

C:\Users\amao>wsl -v
WSL 版本： 1.2.5.0
内核版本： 5.15.90.1
WSLg 版本： 1.0.51
MSRDC 版本： 1.2.3770
Direct3D 版本： 1.608.2-61064218
DXCore 版本： 10.0.25131.1002-220531-1700.rs-onecore-base2-hyp
Windows 版本： 10.0.19045.3086
```
成功安装centos7.9.2009
```bat
C:\Users\amao>wsl -l -v
  NAME              STATE           VERSION
* Ubuntu            Running         2
  centos            Running         2
```
```shell
[root@WG11-0009 amao]# lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch:cxx-4.1-amd64:cxx-4.1-noarch:desktop-4.1-amd64:desktop-4.1-noarch:languages-4.1-amd64:languages-4.1-noarch:printing-4.1-amd64:printing-4.1-noarch
Distributor ID: CentOS
Description:    CentOS Linux release 7.9.2009 (Core)
Release:        7.9.2009
Codename:       Core
```

# 安装docker mongo
* docker引擎官方安装步骤[Install Docker Engine on CentOS](https://docs.docker.com/engine/install/centos/)
* [mongo 的docker版本](https://hub.docker.com/_/mongo)
  1. 如果mongo启动失败，使用`docker logs`排查失败的原因。

成功安装docker
```shell
[root@WG11-0009 amao]# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2024-03-01 12:04:30 CST; 26s ago
     Docs: https://docs.docker.com
 Main PID: 141 (dockerd)
    Tasks: 35
   Memory: 231.8M
   CGroup: /system.slice/docker.service
           └─141 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
成功安装mongo4.4.28
```
[root@WG11-0009 amao]# sudo docker run --name mongodb -d -p 27017:27017 -v $(pwd)/data:/data/db -e MONGO_INITDB_ROOT_USERNAME=user -e MONGO_INITDB_ROOT_PASSWORD=pass mongo:4.4.28

[root@WG11-0009 amao]# docker ps -a
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS                    PORTS                                           NAMES
0d160b11d2e5   mongo:4.4.28   "docker-entrypoint.s…"   20 hours ago   Up 2 seconds              0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   mongodb
81bcaf55bf98   hello-world    "/hello"                 20 hours ago   Exited (0) 20 hours ago
```

<br>	
<br>	
<b>原文:<br>
<https://lizijie.github.io/2024/03/03/wsl2%E6%90%AD%E5%BB%BAcentos-docker-mongo.html>
<br>
作者github:<br>	
<https://github.com/lizijie>	
</b>