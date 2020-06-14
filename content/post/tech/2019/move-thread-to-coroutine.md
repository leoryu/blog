---
categories:
- Tech
date: "2019-02-20T21:27:46+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- go
- docker
title: 协程代替进程：单容器Go语言应用提供多服务的解决方案
---

## 1. Docker容器内运行多个服务

Docker推荐开发者每个容器内运行一个服务/守护进程，以区分关注的区域。因此，并没有针对容器内运行多进程的场景做过多的设计。例如生命周期上，Docker只关注容器内的1号进程，1号进程以外的生命周期需要开发者去维护。

但在实际开发中，因为各种原因开发者们确实会遇到在单个容器内运行多服务的开发场景。目前解决这一问题的通用解决方案大致分为两种：

1. 写shell脚本运行多进程
2. 使用[supervisor](http://www.supervisord.org/)、[foreman](https://github.com/ddollar/foreman)、[goreman](https://github.com/mattn/goreman)等多进程管理程序

但是这些通用解决方案都有一些缺陷，如依赖过多、占据容器内1号进程难以“优雅关闭”、造成资源浪费等。

## 2. 巧用Go语言的协程

Go语言有一个先天优势：可以用成本（开发成本、计算成本）极小的协程来代替线程。

而笔者在学习[Gin框架](https://github.com/gin-gonic/gin)时发现，Go的协程也可以代替进程，从而实现“单进程多服务”的效果。

而如果实现单进程多服务，则可以在几乎没有任何副作用的前提下实现单容器内运行多个服务。

该方案的Gin框架实现的代码可以参考[multiple-service](https://github.com/gin-gonic/gin/blob/master/examples/multiple-service/main.go)，这里笔者以日常使用的[Echo](https://echo.labstack.com/)框架实现一个多web服务，代码如下：

```go
package main

import (
        "log"
        "net/http"
        "time"

        "golang.org/x/sync/errgroup"

        "github.com/labstack/echo"
)

var g errgroup.Group

func main() {
        server1 := &http.Server{
                Addr:         ":8080",
                Handler:      router1(),
                ReadTimeout:  5 * time.Second,
                WriteTimeout: 10 * time.Second,
        }

        server2 := &http.Server{
                Addr:         ":8081",
                Handler:      router2(),
                ReadTimeout:  5 * time.Second,
                WriteTimeout: 10 * time.Second,
        }

        g.Go(func() error {
                return server1.ListenAndServe()
        })

        g.Go(func() error {
                return server2.ListenAndServe()
        })

        if err := g.Wait(); err != nil {
                log.Fatal(err)
        }
}

func router1() *echo.Echo {
        e := echo.New()
        e.GET("/", func(c echo.Context) error {
                return c.String(http.StatusOK, "This is service 1")
        })
        return e
}

func router2() *echo.Echo {
        e := echo.New()
        e.GET("/", func(c echo.Context) error {
                return c.String(http.StatusOK, "This is service 2")
        })
        return e
}
```

确保以上工作完成后，执行下列命令运行程序：

```sh
go run main.go
```

然后在浏览器中分别输入 http://localhost:8080/ 和 http://localhost:8081/ ，可以看到两个服务返回的不同结果。

## 3. 使用Go协程实现多服务的优势

- 由于多个服务只运行一个run-time，计算资源占用率要低于多进程
- 由于是以单一进程运行，容器内生命周期的维护会简单很多
- 不需要编译多个可执行程序，可以减少镜像的大小