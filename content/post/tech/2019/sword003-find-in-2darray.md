---
categories:
- Tech
date: "2019-04-16T19:45:48+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
markup: mmark
math: true
tags:
- algorithm
- go
title: '[Go的算法实现]二维数组中的查找'
---

剑指Offer中问题3的go实现

<!--more-->

## 问题

> 在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
>
> 例如下面的二维数组就是每行、每列都递增排序。如果在这个数组中查找数字7，则返回true；如果查找数字5，由于数组不含有该数字，则返回false。 
>
>$$
>  \begin{matrix}
>   1 & 2 & 8 & 9 \\
>   2 & 4 & 9 & 12 \\
>   4 & 7 & 10 & 13 \\
>   6 & 8 & 11 & 15
>  \end{matrix}
>$$

## 思路

采用分治思想。从右上角开始：

1. 如果等于我们找的数字，返回true；
2. 如果小于我们找的数字，该行可以删除，在新数组中继续找；
3. 如果大于我们找的数字，该列可以删除，在新数组中继续找；
4. 当数组中没有数字时，返回false；

## 特殊情况测试

- 输入数组为空时的情况
- 单个数测试

## 代码实现

```go
package q003

func Answer(inputArray [][]int, expectedNumber int) (isExisted bool) {
	if inputArray == nil || len(inputArray) == 0 {
		return false
	}
	h := len(inputArray)
	w := len(inputArray[0])
	for i, j := 0, w-1; i < h && j >= 0; {
		if inputArray[i][j] == expectedNumber {
			return true
		}
		if inputArray[i][j] < expectedNumber {
			i++
		}
		if inputArray[i][j] > expectedNumber {
			j--
		}
	}
	return
}
```

## 测试用例

```go
package q003

import "testing"

func TestAnswer(t *testing.T) {
	type args struct {
		inputArray     [][]int
		expectedNumber int
	}
	tests := []struct {
		name          string
		args          args
		wantIsExisted bool
	}{
		{
			"Function test 1",
			args{
				inputArray: [][]int{
					[]int{1, 2, 8, 9},
					[]int{2, 4, 9, 12},
					[]int{4, 7, 10, 13},
					[]int{6, 8, 11, 15},
				},
				expectedNumber: 5,
			},
			false,
		},
		{
			"Function test 2",
			args{
				inputArray: [][]int{
					[]int{1, 2, 8, 9},
					[]int{2, 4, 9, 12},
					[]int{4, 7, 10, 13},
					[]int{6, 8, 11, 15},
				},
				expectedNumber: 7,
			},
			true,
		},
		{
			"Empty 1",
			args{
				inputArray:     [][]int{},
				expectedNumber: 0,
			},
			false,
		},
		{
			"Empty 2",
			args{
				inputArray:     nil,
				expectedNumber: 0,
			},
			false,
		},
		{
			"Just one number 1",
			args{
				inputArray: [][]int{
					[]int{1},
				},
				expectedNumber: 0,
			},
			false,
		},
		{
			"Just one number 2",
			args{
				inputArray: [][]int{
					[]int{1},
				},
				expectedNumber: 1,
			},
			true,
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if gotIsExisted := Answer(tt.args.inputArray, tt.args.expectedNumber); gotIsExisted != tt.wantIsExisted {
				t.Errorf("Answer() = %v, want %v", gotIsExisted, tt.wantIsExisted)
			}
		})
	}
}
```

[代码下载地址](https://github.com/leoryu/AGym/tree/master/sword/q003)
