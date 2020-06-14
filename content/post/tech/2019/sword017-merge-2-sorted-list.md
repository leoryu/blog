---
categories:
- Tech
date: "2019-09-01T21:55:00+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]合并两个排序的链表'
---

剑指Offer中问题17的go实现。

<!--more-->

## 问题

> 输入两个单调递增的链表，输出两个链表合成后的链表，当然我们需要合成后的链表满足单调不减规则。
> 
> 样例输入
> 
> 1 3 5 7 9
> 
> 2 4
> 
> 样例输出
> 
> 1 2 3 4 5 7 9

## 思路

新建一个指针作临时存储用。

## 注意点

- 空链表
- 只有一个节点的链表

## 代码实现

```go
package q017

type NodeList struct {
	Value    int
	NextNode *NodeList
}

func Answer(l1, l2 *NodeList) (ml *NodeList) {
	tmpNode := &NodeList{}
	beforHead := tmpNode
	for l1 != nil || l2 != nil {
		if l1 == nil {
			tmpNode.NextNode = l2
			l2 = l2.NextNode
			break
		}
		if l2 == nil {
			tmpNode.NextNode = l1
			l1 = l1.NextNode
			break
		}
		if l1.Value > l2.Value {
			tmpNode.NextNode = l2
			l2 = l2.NextNode
		} else {
			tmpNode.NextNode = l1
			l1 = l1.NextNode
		}
		tmpNode = tmpNode.NextNode
	}
	return beforHead.NextNode
}
```
