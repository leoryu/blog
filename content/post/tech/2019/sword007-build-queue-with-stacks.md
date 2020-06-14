---
categories:
- Tech
date: "2019-05-28T22:50:04+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]用两个栈实现队列'
---

剑指Offer中问题7的go实现。

<!--more-->

## 问题

> 用两个栈实现一个队列，分别实现其appendTail和deleteHead功能。

## 思路

栈有push和pop功能，都是从尾部进行添加和删除。

我们有两个栈时可以用其中一个实现appendTail，另一个实现deleteHead。

调用appendTail时直接对实现该功能的栈进行push即可。

当调用deleteHead时，判断实现该功能的栈是否为空，若为空将另一个栈全部弹出并压入到该栈，此时该栈的top为head。

## 注意点

- 本程序实现利用了container/list包，没有自己实现stack
- 注意空指针

## 代码实现

```go
package q007

import (
	"container/list"
)

type Queue struct {
	appendStack *list.List
	deleteStack *list.List
}

func (q *Queue) AppendTail(value interface{}) {
	q.appendStack.PushBack(value)
}

func (q *Queue) DeleteHead() (value interface{}) {
	if q.deleteStack.Len() == 0 {
		for q.appendStack.Len() != 0 {
			q.deleteStack.PushBack(q.appendStack.Back())
			q.appendStack.Remove(q.appendStack.Back())
		}
	}
	if q.deleteStack.Len() != 0 {
		value = q.deleteStack.Back().Value
		q.deleteStack.Remove(q.deleteStack.Back())
	}
	return
}
```
