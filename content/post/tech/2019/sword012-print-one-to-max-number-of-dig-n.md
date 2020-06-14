---
categories:
- Tech
date: "2019-08-01T21:30:34+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]打印1到最大的N位数'
---

剑指Offer中问题12的go实现。

<!--more-->

## 问题

> 给定一个数字N，打印从1到最大的N位数。

## 思路

该问题可以转换为N个0-9的全排列问题。

由于每个数的确定都是有第N位数字和f(N-1)确定的，可以用递归来实现。

由于有0站位符的存在，需要一个清理函数清理输出。

## 注意点

## 代码实现

- N为0
- N为负数
- N较大

```go
package q012

import (
	"strconv"
)

var results []string

func Answer(n int) []string {
	results = []string{}
	if n <= 0 {
		return results
	}
	for i := 0; i < 10; i++ {
		AddResult(n, strconv.Itoa(i))
	}
	return results[1:]
}

func AddResult(length int, tmpResult string) {
	if len(tmpResult) == length {
		results = append(results, CleanResult(tmpResult))
		return
	}
	for i := 0; i < 10; i++ {
		AddResult(length, tmpResult+strconv.Itoa(i))
	}
}

func CleanResult(input string) (output string) {
	output = input
	for _, v := range []byte(input) {
		if v != []byte("0")[0] {
			break
		}
		output = output[1:]
	}
	return
}
```
