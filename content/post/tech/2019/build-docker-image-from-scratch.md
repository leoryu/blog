---
categories:
- Tech
date: "2019-03-19T20:53:06+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- docker
title: Docker镜像的无中生有：使用scratch制作自定义最小镜像
---

针对如何制作尽量小的镜像问题，笔者在这里介绍一种用”空镜像“制作自定义最小镜像的方案。

<!--more--> 

## 1. 从石头缝里蹦出来的基础镜像？

这里请大家思考一个问题：docker hub上的那些基础镜像，如nginx、alpine、ubuntu，都是怎么来的？

事实上这些镜像都来源与一个空镜像，[scratch](https://docs.docker.com/samples/library/scratch/)。关于scratch的官方解释如下：

> This image is most useful in the context of building base images (such as debian and busybox) or super minimal images (that contain only a single binary and whatever it requires, such as hello-world).
>
> As of Docker 1.5.0 (specifically, docker/docker#8827), FROM scratch is a no-op in the Dockerfile, and will not create an extra layer in your image (so a previously 2-layer image will be a 1-layer image instead).

官方在这里给出了测试用的[hello-world](https://hub.docker.com/_/hello-world/)镜像Dockerfile：

```dockerfile
FROM scratc
COPY hello /
CMD ["/hello"]
```

该Dockerfile制作出来的镜像只有1.84k。

使用scratch空镜像的本质是让你的程序只调用host主机的Linux内核部分的功能，而不依赖容器内的操作环境功能。

由于host主机的Linux内核部分对Docker容器是共享的，因此其scratch空镜像的大小可以认为近似为0。

## 2. 以Go语言为例做一个最小镜像

做最小镜像的关键在于你要完全了解你的程序需要什么，而针对不同的语言，需要到的背景知识和技巧也不同。

这里笔者用较为熟悉的Go语言为例子做一个最小镜像，程序如下：

```go
package main

import (
        "fmt"
)

func main() {
        fmt.Println("Hello")
}
```

### 2.1. 编译中技巧

如果希望你程序编译出来最小，并保证能够在scratch空镜像内运行，则需要在编译中用一些技巧：

```sh
GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -ldflags '-w -s' hello.go
```

下面笔者将一一解析这条命令中的各个部分含义。

#### 2.1.1. GOOS=linux GOARCH=amd64含义

确保编译出来的程序可以运行在你的容器运行环境。笔者的容器运行环境为amd64 linux环境。

#### 2.1.2 CGO_ENABLED=0含义

该参数是确保你用到的C函数库包含到你的Go run-time中，程序运行时以静态方式内部调用

否则由于scratch空镜像内没有C函数库，Go程序动态调用时会出错。

> PS：Go语言调用C函数库出错的现象也会出现在alpine中，这是因为alpine的C函数库是精简版的。

#### 2.1.3. -ldflags '-w -s'含义

这部分参数是精简掉Debug信息，而让编译出来的程序更小。

-w是精简掉[DWARF](http://www.dwarfstd.org/)。

-s是精简掉debug symbol

### 2.2. Dockerfile书写技巧

Dockerfile的书写也需要一定的技巧，这里先给出笔者的Dockerfile：

```dockerfile
FROM scratch

LABEL authors="LEO"

ADD hello /

CMD [ "/hello" ]
```

#### 2.2.1. RUN命令失效

由于scratch空镜像中没有sh或bash，想mkdir、mv等shell命令是无效的。

因此请在镜像外部把文件结构建立好，然后通过ADD或COPY命令拷贝到容器内。

#### 2.2.2. 单层一次到位原则

尽管在使用scratch空镜像时无法使用RUN，不大会违反这一原则，但笔者认为大家有必要知道这一原则。

Dokcer建立镜像时会保留每一层的状态，这就导致当我们在Dockerfile写下下列命令时：

```dockerfile
RUN mv /here/{1M文件} /there/{1M文件}
```

制作出来的镜像会大出1M，这是因为/here下的文件并没有因为mv命令而消失，它永远保存在RUN命令之前的那一层中了，RUN命令这一层只是把它藏了起来。

### 2.3. 制作镜像查看成果

在包含Dockerfile和hello程序的位置执行下面的命令制作镜像：

```sh
docker build -t hello:latest .
```

执行下面的命令验证容器是否能正常运行：

```sh
docker run hello:latest
```

现在看一下我们镜像的大小和可执行程序的大小：

```sh
$ docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
hello                   latest              116f651aa3b5        6 seconds ago       356.2 kB

$ ll hello
-rwxr-xr-x 1 ... ... ... 348K Nov 12 18:51 hello
```

可以看到我们最终得到的镜像的大小就接近与可执行文件的大小。

## 3. 总结

使用scratch空镜像的优点：

- 真正意义上的最小镜像，镜像大小约等于执行文件大小
- 安全稳定，只需要关注维护程序本身和Linux内核安全更新即可
- 最高效的资源利用，容器内没有任何多余程序或服务占用资源
- 制作镜像方便快捷，由于scratch空镜像不需要load或pull就能使用，流水线上制作镜像更加方便快捷

缺点：

- 由于没有sh或bash，无法进入容器内进行交互式调试

> PS：该问题实际可以通过构建自定义基础镜像解决。但笔者个人认为生产环境应该通过日志分析问题，而不是进入到容器内进行调试。