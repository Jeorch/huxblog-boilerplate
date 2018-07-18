---
layout:     post
title:      "Docker背后的内核知识-namespace-cgroups"
subtitle:   " \"Docker namespace\""
date:       2018-07-17 18:00:00
author:     "Jeorch"
header-img: "img/post-bg-docker.jpg"
catalog: true
tags:
    - Docker
    - Namespace
---

> “Yeah It's Docker. ”


## 前言

最近在看《Docker容器与容器云》，目前看到了3.1.Docker背后的内核知识：
  1. 利用namespace实现资源隔离
  2. 利用cgroups实现资源隔离

## 利用namespace实现资源隔离
*有兴趣的可以进到传送门里，敲一敲c代码，自行尝试，PS：不要敲错引入的库名，在zsh下尝试效果更佳*
  - UTS —— 主机名与域名 <br/>[传送门](http://dockone.io/article/76)
  - IPC —— 信号量、消息队列和共享内存  <br/>[传送门](http://dockone.io/article/79)
  - PID —— 进程编号 <br/>[传送门](http://dockone.io/article/81)
  - Network —— 网络设备、网络栈、端口等 <br/>[传送门](http://dockone.io/article/83)
  - Mount —— 挂载点（文件系统）  <br/>[传送门](http://dockone.io/article/82)
  - User —— 用户和用户组  <br/>[传送门](https://yq.aliyun.com/articles/30349)

## 利用cgroups实现资源隔离

- 资源限制
- 优先级分配
- 资源统计
- 任务控制
