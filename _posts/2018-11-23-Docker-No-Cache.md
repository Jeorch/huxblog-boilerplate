---
layout:     post
title:      "docker build高效利用cache"
subtitle:   " \"docker build --no-cache\""
date:       2018-11-23 18:00:00
author:     "Jeorch"
header-img: "img/post-bg-docker.jpg"
catalog: true
tags:
    - Docker
---

> “Yeah It's Docker. ”


## 前言
Dockerfile 可以通过docker build命令构建为一个新的镜像，Dockerfile 中每一条命令都会构建出一个新的镜像层。当你重新build相同的Docker时，Docker会逐条语句check自身的cache镜像层，如果命中相同的，就使用cache而不执行这条语句继续往下逐条check直至build完成。Docker build cache相关知识可以参考[docker build 的 cache 机制](https://blog.csdn.net/jeorch/article/details/84402089)。

本文主要讲述如何利用cache提高build的效率。

## 正文

#### 1. build Dockerfile 部分使用 cache
目前 docker 利用 cache 的基本原理是在父节点存在 cache 的前提下 当前 dockerfile 的那条语句之前也构建过就可以用 cache。例外就是 ADD 和 COPY 需要计算一个 checksum。
大致意思就是如果你中间有一层不能用 cache 那之后的层次就都不能用了。因此主要功夫在写 dockerfile 上，尽量把不变的内容放在前面，频繁变化的内容放在最后。简单说就是 把 ADD 和 COPY 的内容放最后一句，这样很多情况下只有最后一层变，前面都能用 cache。
比如我们在Dockerfile中使用了RUN go get xxx，我们提交了一下最新代码，然后重新build时，会命中cache，导致build出的image使用的还是之前的代码。
```
▶ docker build -t="jeorch/ddsaas:1.0.1" ./
Sending build context to Docker daemon  266.2kB
Step 1/8 : FROM golang:alpine
 ---> 20ff4d6283c0
Step 2/8 : RUN apk add --no-cache git mercurial
 ---> Using cache
 ---> 7c76541453d4
Step 3/8 : RUN go get github.com/alfredyang1986/blackmirror
 ---> Using cache
 ---> b8a5b7464bed
Step 4/8 : RUN go get github.com/alfredyang1986/ddsaas
 ---> Using cache
 ---> 27e569d1cdf5
Step 5/8 : ADD deploy-config/ /go/bin/
 ---> Using cache
 ---> 0a72ad21b21c
Step 6/8 : RUN go install -v github.com/alfredyang1986/ddsaas
 ---> Using cache
 ---> beda69880a63
Step 7/8 : WORKDIR /go/bin
 ---> Using cache
 ---> f57778461a26
Step 8/8 : ENTRYPOINT ["ddsaas"]
 ---> Using cache
 ---> 4c41a3140c4d
Successfully built 4c41a3140c4d
Successfully tagged jeorch/ddsaas:1.0.1
```
我们可以在Dockerfile中的RUN go get xxx之前添加一句标识来阻断cache继续命中，这样接下来的语句都会重新执行而不继续check cache。我在Dockerfile中加入了一行“MAINTAINER Jeorch”，Step-2还会使用cache，而接下来的都不会使用cache。
```
▶ docker build -t="jeorch/ddsaas:1.0.1" ./
Sending build context to Docker daemon  266.2kB
Step 1/9 : FROM golang:alpine
 ---> 20ff4d6283c0
Step 2/9 : RUN apk add --no-cache git mercurial
 ---> Using cache
 ---> 7c76541453d4
Step 3/9 : MAINTAINER Jeorch
 ---> Running in 33bbfbd73508
Removing intermediate container 33bbfbd73508
 ---> ac2188cd26c4
Step 4/9 : RUN go get github.com/alfredyang1986/blackmirror
 ---> Running in 3363c4394984
Removing intermediate container 3363c4394984
 ---> db9237361148
Step 5/9 : RUN go get github.com/alfredyang1986/ddsaas
 ---> Running in 738534a17f5e
Removing intermediate container 738534a17f5e
 ---> fe40c81e076d
Step 6/9 : ADD deploy-config/ /go/bin/
 ---> ddad378cdf99
Step 7/9 : RUN go install -v github.com/alfredyang1986/ddsaas
 ---> Running in 74d83f8a7c9a
Removing intermediate container 74d83f8a7c9a
 ---> 50f17bf6530e
Step 8/9 : WORKDIR /go/bin
 ---> Running in a904feb924d2
Removing intermediate container a904feb924d2
 ---> f1297882f28d
Step 9/9 : ENTRYPOINT ["ddsaas"]
 ---> Running in 3dda764560b3
Removing intermediate container 3dda764560b3
 ---> 236b15602a89
Successfully built 236b15602a89
Successfully tagged jeorch/ddsaas:1.0.1
```
LABEL比MAINTAINER更灵活，推荐使用LABEL，弃用MAINTAINER。详情看[深入Dockerfile（一）: 语法指南](https://github.com/qianlei90/Blog/issues/35)
#### 2. build Dockerfile 不使用cache
Dockerfile需要全部重新build，即我们不需要利用cache，我们可以在docker build 命令后使用--no-cache。
```
▶ docker build -t="jeorch/ddsaas:1.0.1" ./ --no-cache
Sending build context to Docker daemon  266.2kB
Step 1/8 : FROM golang:alpine
 ---> 20ff4d6283c0
Step 2/8 : RUN apk add --no-cache git mercurial
 ---> Running in 770bcf003ff2
fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.8/community/x86_64/APKINDEX.tar.gz
(1/16) Installing nghttp2-libs (1.32.0-r0)
(2/16) Installing libssh2 (1.8.0-r3)
(3/16) Installing libcurl (7.61.1-r1)
(4/16) Installing expat (2.2.5-r0)
(5/16) Installing pcre2 (10.31-r0)
(6/16) Installing git (2.18.1-r0)
(7/16) Installing libbz2 (1.0.6-r6)
(8/16) Installing libffi (3.2.1-r4)
(9/16) Installing gdbm (1.13-r1)
(10/16) Installing ncurses-terminfo-base (6.1_p20180818-r1)
(11/16) Installing ncurses-terminfo (6.1_p20180818-r1)
(12/16) Installing ncurses-libs (6.1_p20180818-r1)
(13/16) Installing readline (7.0.003-r0)
(14/16) Installing sqlite-libs (3.24.0-r0)
(15/16) Installing python2 (2.7.15-r1)
(16/16) Installing mercurial (4.6.1-r0)
Executing busybox-1.28.4-r0.trigger
OK: 80 MiB in 30 packages
Removing intermediate container 770bcf003ff2
 ---> 86dfeb5c30c2
Step 3/8 : RUN go get github.com/alfredyang1986/blackmirror
 ---> Running in d7dd1c8fab79
Removing intermediate container d7dd1c8fab79
 ---> c7b61e4b6722
Step 4/8 : RUN go get github.com/alfredyang1986/ddsaas
 ---> Running in c783e5fe874c
Removing intermediate container c783e5fe874c
 ---> c07eda8eb619
Step 5/8 : ADD deploy-config/ /go/bin/
 ---> aa51cfe6311f
Step 6/8 : RUN go install -v github.com/alfredyang1986/ddsaas
 ---> Running in 6ca73a6969df
Removing intermediate container 6ca73a6969df
 ---> be7090e0095e
Step 7/8 : WORKDIR /go/bin
 ---> Running in 9f19f951d662
Removing intermediate container 9f19f951d662
 ---> a0894151c5e6
Step 8/8 : ENTRYPOINT ["ddsaas"]
 ---> Running in 9404dad6e3bf
Removing intermediate container 9404dad6e3bf
 ---> 2681d9a56d7b
Successfully built 2681d9a56d7b
Successfully tagged jeorch/ddsaas:1.0.1
```

#### 3. build Dockerfile 全部利用cache
正常执行docker build就行。

## 参考
 - [镜像构建时，如何更充分利用缓存](http://dockone.io/question/618)
 - [Dockerfile构建过程-去除缓存构建](https://blog.csdn.net/shenzhen_zsw/article/details/74121794)
 - [Docker实践(25) – 不使用缓存重建镜像](https://www.centos.bz/2016/11/rebuild-docker-image-without-cache/)
