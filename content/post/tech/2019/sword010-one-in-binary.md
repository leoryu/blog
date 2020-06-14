---
categories:
- Tech
date: "2019-07-17T21:40:24+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]二进制中1的个数'
---

剑指Offer中问题10的go实现。

<!--more-->

## 问题

> 输入一个整数，输出该数二进制表示中1的个数。

## 思路

解决该问题需要使用到位运算。

1. 首先将数字减去1
2. 之后将该数字与减去1的数字进行位运算，结果覆盖该数字，并向计数器加1
3. 重复步骤2，直到该数字为0停止

## 注意点

- 输入负数
- 输入0
- 输入MaxInt64和MinInt64

## 代码实现

```go
package q010

func Answer(checkedNum int) (numOfOne int) {
	if checkedNum < 0 {
		checkedNum = 0 - checkedNum
	}
	for checkedNum != 0 {
		checkedNum = checkedNum & (checkedNum - 1)
		numOfOne++
	}
	return
}
```
