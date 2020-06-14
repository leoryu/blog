---
categories:
- Tech
date: "2019-05-27T22:28:04+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]重建二叉树'
---

剑指Offer中问题6的go实现。

<!--more-->

## 问题

> 输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如某二叉树的前序遍历序列{1,2,4,7,3,5,6,8}，中序遍历序列{4,7,2,1,5,3,8,6}。

## 思路

注意有一个特殊条件是遍历结果中不含重复数字。

前序遍历顺序为根左右，中序遍历顺序为左根右。根据前面的不重复条件，我们可以找到中序遍历中的根节点位置，并确定左右树的集合。

之后分别在左右树中递归调用函数即可。

## 特殊情况测试

- 注意程序中返回值，root，默认情况为nil；
- 空树的情况；
- 树中的每个节点只有左（右）节点的情况；

## 代码实现

```go
package q006

type BinaryTreeNode struct {
	Value     int
	LeftNode  *BinaryTreeNode
	RightNode *BinaryTreeNode
}

func AnswerWithRecursion(preOrder []int, inOrder []int) (root *BinaryTreeNode) {
	if preOrder == nil || len(preOrder) == 0 {
		return
	}
	root = new(BinaryTreeNode)
	root.Value = preOrder[0]
	var rootPosition int
	for i, v := range inOrder {
		if v == root.Value {
			rootPosition = i
		}
	}
	// leftLen是多余的，但有助于理解
	leftLen := rootPosition
	rightLen := len(preOrder) - rootPosition - 1
	if leftLen > 0 {
		root.LeftNode = AnswerWithRecursion(preOrder[1:rootPosition+1], inOrder[:rootPosition])
	}
	if rightLen > 0 {
		root.RightNode = AnswerWithRecursion(preOrder[rootPosition+1:], inOrder[rootPosition+1:])
	}
	return
}
```

## 测试用例

```go
package q006

import (
	"reflect"
	"testing"
)

var (
	node1 *BinaryTreeNode
	node2 *BinaryTreeNode
)

func init() {
	node1 = new(BinaryTreeNode)
	node1.Value = 1
	node1.LeftNode = new(BinaryTreeNode)
	node1.LeftNode.Value = 2
	node1.LeftNode.LeftNode = new(BinaryTreeNode)
	node1.LeftNode.LeftNode.Value = 4
	node1.LeftNode.LeftNode.RightNode = new(BinaryTreeNode)
	node1.LeftNode.LeftNode.RightNode.Value = 7
	node1.RightNode = new(BinaryTreeNode)
	node1.RightNode.Value = 3
	node1.RightNode.LeftNode = new(BinaryTreeNode)
	node1.RightNode.LeftNode.Value = 5
	node1.RightNode.RightNode = new(BinaryTreeNode)
	node1.RightNode.RightNode.Value = 6
	node1.RightNode.RightNode.LeftNode = new(BinaryTreeNode)
	node1.RightNode.RightNode.LeftNode.Value = 8
}

func TestAnswerWithRecursion(t *testing.T) {
	type args struct {
		preOrder []int
		inOrder  []int
	}
	tests := []struct {
		name     string
		args     args
		wantRoot *BinaryTreeNode
	}{
		{
			"Function test 1",
			args{
				preOrder: []int{1, 2, 4, 7, 3, 5, 6, 8},
				inOrder:  []int{4, 7, 2, 1, 5, 3, 8, 6},
			},
			node1,
		},
		{
			"Empty",
			args{
				preOrder: []int{},
				inOrder:  []int{},
			},
			nil,
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if gotRoot := AnswerWithRecursion(tt.args.preOrder, tt.args.inOrder); !reflect.DeepEqual(gotRoot, tt.wantRoot) {
				t.Errorf("AnswerWithRecursion() = %v, want %v", gotRoot, tt.wantRoot)
			}
		})
	}
}
```

[代码下载地址](https://github.com/leoryu/AGym/tree/master/sword/q005)
