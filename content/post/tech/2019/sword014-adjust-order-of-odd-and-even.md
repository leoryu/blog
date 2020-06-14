---
categories:
- Tech
date: "2019-08-02T23:30:34+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]调整数组顺序使奇数位于偶数前面'
---

剑指Offer中问题14的go实现。

<!--more-->

## 问题

> 输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。

## 思路

使用冒泡法可以解决问题，且能保证顺序不变，但是计算复杂度略高。

为减少复杂度可以使用两个指针从前后分别开始向中间靠拢，一旦发现偶数在奇数的前面就交换两个数字的位置，不过这种方法会影响数字之间的相对位置。

## 注意点

- 空数组
- 全偶/奇数组

## 代码实现

```go
package q014

func AnswerBubble(inputList []int) (outputList []int) {
	head := 0
	tail := len(inputList) - 1
	for head <= tail {
		for inputList[head]%2 == 1 && head <= tail {
			head++
		}
		for i, j := head, head+1; j < len(inputList) && head <= tail; {
			inputList[i], inputList[j] = inputList[j], inputList[i]
			i++
			j++
		}
		tail--
	}
	return inputList
}

func AnswerTwoPointer(inputList []int) (outputList []int) {
	head := 1
	tail := len(inputList) - 1
	for head < tail {
		for inputList[head]%2 == 1 && head < tail {
			head++
		}
		for inputList[tail]%2 == 0 && head < tail {
			tail--
		}
		inputList[head], inputList[tail] = inputList[tail], inputList[head]
	}
	return inputList
}
```
