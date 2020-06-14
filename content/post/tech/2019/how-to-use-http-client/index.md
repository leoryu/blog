---
categories:
- Tech
date: "2019-07-24T23:40:24+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- go
title: 高并发场景Golang进行转发和代理时的注意事项
---

在以RESTful API通信为基础的微服务场景下，Golang自带的HTTP客户端和反向代理在调用外部服务时非常好用。但是如果使用不当，程序经常会出现一些奇怪的错误，并且这些错误只会在高并发的场景下才会显露出来，在程序进入实际生产环境前很难被发现。

笔者将在本文中将根据工作中遇到的一些问题，介绍一下高并发场景下Golang进行转发和代理时需要注意的事项。由于代理的本质和HTTP Client在Golang中是一样的，了解其一便可，所以本文将只针对HTTP Client进行讨论。

<!--more-->

## 准备工作

在进入正题之前我们在这里需要先设计一个简单的测试服务器，用于和HTTP Client进行交互用。代码如下：

```go
package main

import (
	"flag"
	"log"
	"net/http"
	"runtime"
)

func main() {
    // 限制程序使用的逻辑处理器数目
	runtime.GOMAXPROCS(1)
	port := flag.String("port", "12345", "Listen port")
	flag.Parse()
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // 休眠一段时间模拟业务处理
		time.Sleep(300 * time.Millisecond)
		w.WriteHeader(http.StatusNoContent)
	})
	log.Println("Listen port: " + *port)
	log.Fatal(http.ListenAndServe(":"+*port, nil))
}
```

该服务会休眠一段时间模拟业务处理，之后返回204。

## 以单例模式创建HTTP Client

没有使用单例模式创建HTTP Client是最为常见的造成高并发能力下降的原因。例如下面的这段代码：

```go
package main

import (
	"flag"
	"log"
	"net/http"
	"runtime"
)

func main() {
	runtime.GOMAXPROCS(1)
	port := flag.String("port", "54321", "Listen port")
	tPort := flag.String("tport", "12345", "Target port")
	flag.Parse()
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        re, _ := http.NewRequest(http.MethodGet, "http://127.0.0.1:"+*tPort, nil)
        // 在协程中创建一个新的Client
		c := &http.Client{}
		_, err := c.Do(re)
		if err != nil {
			w.WriteHeader(http.StatusBadGateway)
			log.Println(err)
			return
		}
		w.WriteHeader(http.StatusNoContent)
	})
	log.Println("Listen port: " + *port)
	log.Fatal(http.ListenAndServe(":"+*port, nil))
}
```

上面的这段代码会造成每一次请求都会创建一个HTTP Client，而每个HTTP Client都会占用维护一份独立的连接池。这在短时间内高并发产生时,IO不会合并会造成网络资源耗尽，经常出现“cannot assign requested address”的错误，并造成CPU占用率居高不下。

其实从官方文档中我们可以了解到golang自身的HTTP Client是线程安全的，因此正确使用HTTP Client的方式是使用单例模式去创建它。这里我们简单的在程序初始化时创建一个全局单例HTTP Client来解决这一问题：

```go
package main

import (
	"flag"
	"log"
	"net/http"
	"runtime"
)

// 初始化时创建一个全局单例
var c = &http.Client{}

func main() {
	runtime.GOMAXPROCS(1)
	port := flag.String("port", "54321", "Listen port")
	tPort := flag.String("tport", "12345", "Target port")
	flag.Parse()
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        re, _ := http.NewRequest(http.MethodGet, "http://127.0.0.1:"+*tPort, nil)
        // 由于HTTP Client是线程安全的，可以直接在协程中引用它进行操作
		_, err := c.Do(re)
		if err != nil {
			w.WriteHeader(http.StatusBadGateway)
			log.Println(err)
			return
		}
		w.WriteHeader(http.StatusNoContent)
	})
	log.Println("Listen port: " + *port)
	log.Fatal(http.ListenAndServe(":"+*port, nil))
}
```

## 及时关闭获取到的Body

如果业务逻辑中需要获取其他服务响应的Body时，就需要注意避免发生内存泄漏的问题。由于HTTP Client获取相应的Body是一个ReadCloser，如果不主动关闭，Body会一直维持打开状态，从而整个协程及协程内局部变量不会被回收。高并发时很快内存就会被耗尽导致服务崩溃。

为避免这一问题出现需要在读取完Body体后第一时间将Body关闭，如果不需要读取Body建议将Body输出到一个io.Discard后再第一时间关闭。具体可以参考下面的代码：

```go
package main

import (
	"flag"
	"io"
	"io/ioutil"
	"log"
	"net/http"
	"runtime"
)

var c = &http.Client{}

func main() {
	runtime.GOMAXPROCS(1)
	port := flag.String("port", "54321", "Listen port")
	tPort := flag.String("tport", "12345", "Target port")
	flag.Parse()
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		re, _ := http.NewRequest(http.MethodGet, "http://127.0.0.1:"+*tPort, nil)
		_, err := c.Do(re)
		if err != nil {
			w.WriteHeader(http.StatusBadGateway)
			log.Println(err)
			return
        }
        // 在不需Body时将Body读取输出到Discard
        io.Copy(ioutil.Discard, re.Body)
        // 读取动作完成后第一时间关闭Body
		re.Body.Close()
		w.WriteHeader(http.StatusNoContent)
	})
	log.Println("Listen port: " + *port)
	log.Fatal(http.ListenAndServe(":"+*port, nil))
}
```

这里不是很推荐使用defer来关闭Body，因为使用defer会造成不必要的资源浪费。

## 管理好连接池

在上面的几节中，使用的HTTP Client使用的都是默认的连接池配置。默认配置中的MaxIdleConnsPerHost需要注意下，因为它的默认值是2，如果转发的目标地址很少，建议将其修改为与MaxIdleConns一致：

```go
var c = &http.Client{
	Transport: &http.Transport{
        MaxIdleConnsPerHost: 100,
        // MaxIdleConns的默认值为100
		// MaxIdleConns:        100,
	},
}
```
