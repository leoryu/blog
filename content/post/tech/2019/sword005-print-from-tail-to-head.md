---
categories:
- Tech
date: "2019-05-12T22:18:39+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]从尾到头打印链表'
---

剑指Offer中问题5的go实现。

<!--more-->

## 问题

> 输入一个链表的头节点，从尾到头反过来打印出每个节点的值。

## 思路

使用递归，从头节点开始，以下一个节点为入参递归调用，直到入参为nil为止。

## 注意点

- 注意入参为空指针情况;
- 使用递归虽然代码优雅，但效率低，可以考虑引入一个容器而使用循环实现;

## 代码实现

```go
package q005

type NodeList struct {
	Value    int
	NextNode *NodeList
}

func AnswerWithRecursion(head *NodeList) (result []int) {
	if head == nil {
		return []int{}
	}
	nextResult := AnswerWithRecursion(head.NextNode)
	if len(nextResult) != 0 {
		return append(nextResult, head.Value)
	}
	return []int{head.Value}
}

func AnswerWithoutRecursion(head *NodeList) (result []int) {
	count := 0
	node := head
	for node != nil {
		count++
		node = node.NextNode
	}
	result = make([]int, count)
	for head != nil {
		result[count-1] = head.Value
		head = head.NextNode
		count--
	}
	return
}
```

## 测试用例

```go
package q005

import (
	"reflect"
	"testing"
)

func TestAnswerWithRecursion(t *testing.T) {
	type args struct {
		head *NodeList
	}
	tests := []struct {
		name       string
		args       args
		wantResult []int
	}{
		{
			"Function test 1",
			args{
				head: &NodeList{
					Value: 1,
					NextNode: &NodeList{
						Value: 2,
						NextNode: &NodeList{
							Value:    3,
							NextNode: nil,
						},
					},
				},
			},
			[]int{3, 2, 1},
		},
		{
			"Function test 2",
			args{
				head: &NodeList{
					Value:    1,
					NextNode: nil,
				},
			},
			[]int{1},
		},
		{
			"Empty",
			args{
				head: nil,
			},
			[]int{},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if gotResult := AnswerWithRecursion(tt.args.head); !reflect.DeepEqual(gotResult, tt.wantResult) {
				t.Errorf("AnswerWithRecursion() = %v, want %v", gotResult, tt.wantResult)
			}
		})
	}
}

func TestAnswerWithoutRecursion(t *testing.T) {
	type args struct {
		head *NodeList
	}
	tests := []struct {
		name       string
		args       args
		wantResult []int
	}{
		{
			"Function test 1",
			args{
				head: &NodeList{
					Value: 1,
					NextNode: &NodeList{
						Value: 2,
						NextNode: &NodeList{
							Value:    3,
							NextNode: nil,
						},
					},
				},
			},
			[]int{3, 2, 1},
		},
		{
			"Function test 2",
			args{
				head: &NodeList{
					Value:    1,
					NextNode: nil,
				},
			},
			[]int{1},
		},
		{
			"Empty",
			args{
				head: nil,
			},
			[]int{},
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if gotResult := AnswerWithoutRecursion(tt.args.head); !reflect.DeepEqual(gotResult, tt.wantResult) {
				t.Errorf("AnswerWithoutRecursion() = %v, want %v", gotResult, tt.wantResult)
			}
		})
	}
}
```

[代码下载地址](https://github.com/leoryu/AGym/tree/master/sword/q005)