---
categories:
- Tech
date: "2019-08-30T22:00:34+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]链表中倒数第k个结点'
---

剑指Offer中问题15的go实现。

<!--more-->

## 问题

> 输入一个链表，输出该链表中倒数第k个节点。

## 思路

使用两个指针，两个指针间隔k，当第一个指针到达尾部时，第二指针的位置就是要找的节点位置

## 注意点

- 空链表
- 链表长度不足k

## 代码实现

```go
package q015

type NodeList struct {
	Value    int
	NextNode *NodeList
}

func Answer(head *NodeList, k int) (target *NodeList) {
	if head == nil || k < 1 {
		return
	}
	target = head
	for k > 1 && head != nil {
		head = head.NextNode
		k--
	}
	if head == nil {
		return nil
	}
	for head.NextNode != nil {
		head = head.NextNode
		target = target.NextNode
	}
	return target
}
```
