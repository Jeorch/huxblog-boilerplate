---
layout:     post
title:      "安装最新版Docker"
subtitle:   " \"Docker latest\""
date:       2018-07-25 18:00:00
author:     "Jeorch"
header-img: "img/post-bg-docker.jpg"
catalog: true
tags:
    - Docker
    - install
    - latest
---

> “Yeah It's Docker. ”


## 前言

最近在CentOS 7上安装了docker后，用了一阵子后，发现docker版本不是最新的，用的yum安装的，并且安装前进行了yum update，结果安装后的docker版本是1.13.x的，而最新版本已经到了18.06了。

## 正文

#### 1. 卸载docker旧版本(***如果没安装过docker可以掠过此步骤***)
```
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine
```
#### 2. 安装相关工具类
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
#### 3. 添加aliyun国内源配置docker仓库：
```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
#### 4. 安装最新发布版的docker
***需要到官网进行确认目前的版本，v1.13.x及之前的是“docker”，目前v18.x是“docker ce”***
```
sudo yum install docker-ce
```
#### 5. 开启docker服务
```
sudo systemctl start docker.service
```
#### 6. 允许docker服务开机自启动
```
sudo systemctl enable docker.service
```
#### 7. 查看安装的docker版本
```
docker --version
```
#### 8. 查看安装的docker信息
```
docker info
```
## 参考
 - (Centos环境docker的正确安装及疑难杂症)[http://www.mamicode.com/info-detail-2342582.html]
