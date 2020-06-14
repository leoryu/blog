---
categories:
- Tech
date: "2019-07-26T22:40:34+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]数值的整数次方'
---

剑指Offer中问题11的go实现。

<!--more-->

## 问题

> 给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。

## 思路

result(n) = result(n-1) *or/ result

使用递归即可。

## 注意点

- base为0
- base为负值
- exponent为0
- exponent为负值

## 代码实现

```go
package q011

func Answer(base float64, exponent int) (result float64) {
	if exponent == 0 {
		return 1
	}
	if exponent == 1 {
		return base
	}
	if exponent == -1 {
		return 1 / base
	}
	result = Answer(base, exponent>>1)
	result *= result
	if (exponent & 1) == 1 {
		result *= base
	}
	return
}
```
