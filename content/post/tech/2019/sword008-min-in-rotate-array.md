---
categories:
- Tech
date: "2019-07-08T22:50:04+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]旋转数组的最小数字'
---

剑指Offer中问题8的go实现。

<!--more-->

## 问题

>把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增序列的一个旋转，输出旋转数组的最小元素。例如
>
>>数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转。
>
>该数组的最小值为1。

## 思路

该问题是为了找到后半段的首位值，可以使用二分查找法。

但是需要将数组分成前后两段。

当二分查找取中间值时，该值根据处于这两段中的哪一段来分情况分析：

1. 当该值位于前半段时，该值>=旋转数组的首位值
2. 当该值位于后半段时，该值<=旋转数组的首位值

## 注意点

- 当旋转数组的首位值、末尾值以及中位值全部相等无法判断中位点是位于前半段还是后半段
- 测试用例注意特殊输入：空数组、只有一位的数组、全等数组

## 代码实现

```go
package q008

func Answer(input []int) (output []int) {
	if input == nil || len(input) == 0 {
		return []int{0}
	}
	if len(input) == 1 {
		return input
	}
	lastPoint := len(input) - 1
	middlePoint := len(input) / 2
	if input[lastPoint] > input[0] {
		return input[:1]
	}
	if input[0] == input[middlePoint] && input[0] == input[lastPoint] {
		return Answer(input[1:])
	}
	if input[middlePoint] >= input[0] {
		return Answer(input[middlePoint+1:])
	}
	return Answer(input[1 : middlePoint+1])
}
```