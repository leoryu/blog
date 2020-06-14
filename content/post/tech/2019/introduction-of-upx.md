---
categories:
- Tech
date: "2019-01-19T19:40:24+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- go
- docker
title: 压缩界“黑科技”UPX：把可执行文件压缩至原有的30%
---

笔者在这里介绍一个工具，UPX，该工具可以在不影响程序运行时功能和性能的前提下，将可执行文件压缩至原体积的30%-50%。

<!--more--> 

## 1. UPX的使用及效果

### 1.1. 下载安装

目前[UPX](https://github.com/upx/upx)项目已经迁移到Github平台托管，可以在其项目的[Release](https://github.com/upx/upx/releases)中下载到对应平台的编译版本。这里以upx-3.95-amd64_linux.tar.xz为例子进行安装。

在终端中依次执行下面的命令：

```sh
xz -d upx-3.95-amd64_linux.tar.xz
tar -xvf upx-3.95-amd64_linux.tar
sudo mv upx-3.95-amd64_linux/upx /usr/bin/upx
```
执行完后可以输入upx -v来验证是否安装成功。


### 1.2. 使用效果

这里我们使用Go编写一个Hello程序来演示UPX的效果，代码如下：

```golang
package main

import (
        "fmt"
)

func main() {
        fmt.Println("Hello")
}
```

执行”go build hello.go“后编译出的可执行文件大小如下：

```sh
$ ll
total 1.5M
-rwxr-xr-x 1 ... ... ... 1.5M Nov  9 14:48 hello
-rw-r--r-- 1 ... ... ...   71 Nov  9 14:36 hello.go
```

执行”upx hello“对其压缩，大小如下：

```sh
$ ll
total 500K
-rwxr-xr-x 1 ... ... ... 495K Nov  9 14:48 hello
-rw-r--r-- 1 ... ... ...   71 Nov  9 14:36 hello.go
```

**我们的Hello程序从原来的1.5M变成了495K，压缩率为32.65%。**

## 2. UPX工作原理

UPX的本质与tar和7-zip是一样的，就是一个压缩工具。但是UPX的神奇之处在于解压时根本不需要工具，在程序执行的开始自动把程序解压出来，放到内存中。

UPX拥有这一能力的关键在于upx stub，下面是UPX压缩和解压的流程：

### 2.1. 压缩过程

- upx通过压缩算法对文件进行压缩
- upx在外部加上一层”壳“
- upx把upx stub放入到”壳“中

### 2.2. 解压过程

- 用户触发运行程序
- upx stub”指挥“CPU剥掉”壳“解压出程序
- 程序运行

通过上面的过程可以了解到upx stub的关键地位，但是upx stub结合UPX项目的GPL协议让这个工具能否商用产生疑问。

## 3. UPX license解读

首先说结论，UPX工具是可以用于压缩商业软件的。

UPX的授权协议选择了”病毒式“GPL协议。根据前面我们介绍的UPX工作原理，如果没有额外规定，按理说包含upx stub的被压缩程序也需要符合GPL协议。

但是UPX的license中添加了[SPECIAL EXCEPTION FOR COMPRESSED EXECUTABLES](https://upx.github.io/upx-license.html)，让被压缩程序无需符合GPL协议，并明确声明可以用于压缩商业软件。

这里笔者也引用了两处开发社区对upx能否用于商业软件的解读，其结论与笔者上面的说明结论一致，这里笔者不再进行翻译。

[stackoverflow社区开发者](https://stackoverflow.com/questions/791075/can-i-use-upx-packer-to-compress-a-commercial-program/)

> Yes, you can. Please have a look at the UPX license. There's a special exception for commercial applications so they can use UPX.
Note to Zifre: UPX is GPL, but it's not the same as creating an image with GIMP, since part of the UPX code is added to the commercial application (the decompressing part). That's why the exception in the license is required.


[日本开发者](http://tdoc.info/blog/2016/03/01/go_diet.html)

> ライセンスについて。UPXはGPLに例外事項をつけたライセンスをとっています。これは、GPLであるUPXがバイナリにリンクする形となるため、圧縮対象となるバイナリも通常はGPLになる必要があるのですが、リンクするUPXのコードを改変しないかぎりその必要はない、という例外条項です。ライセンスを確認していただけると分かるように、商用プログラムに対して適用できると明確に書かれています。

## 4. 总结

- UPX能将可执行文件压缩至原体积的30%-50%
- 由于UPX只是在程序启动时多了一步解压过程，因此对运行时的功能和性能没有影响
- UPX是可以用于压缩商业软件
- 尽管可以商用，但请注意UPX的GPL协议，在商用时切忌修改其代码，或发布版本中携带UPX工具