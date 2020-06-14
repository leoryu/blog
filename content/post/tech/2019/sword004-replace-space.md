---
categories:
- Tech
date: "2019-04-27T22:03:01+08:00"
draft: false
image:
  caption: ""
  focal_point: ""
tags:
- algorithm
- go
title: '[Go的算法实现]替换空格'
---

剑指Offer中问题4的go实现

<!--more-->

## 问题

> 请实现一个函数，将一个字符串中的空格替换成“%20”。 例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

## 思路

使用两个指针遍历一次减少复杂度。

1. 遍历一遍字符串计算需要替换的次数；
2. 根据需要替换的次数新建或扩容容器；
3. 使用两个指针分别指向新旧容器的末尾；
4. 将旧容器末尾内容依次写入新容器末尾，遇到需要替换的内容时进行替换；

## 特殊情况测试

- 空字符输入测试

## 代码实现

```go
package q004

import "strings"

func Answer(inputStr string) (outputStr string) {
	old := " "
	new := "%20"
	replaceCount := strings.Count(inputStr, " ")
	outputByte := make([]byte, len(inputStr)+replaceCount*(len(new)-len(old)))
	i := len(inputStr) - 1
	j := len(outputByte) - 1
	for i > -1 {
		if inputStr[i] == old[0] {
			copy(outputByte[j-(len(new)-len(old)):j+1], new)
			j -= len(new) - len(old)
		} else {
			outputByte[j] = inputStr[i]
		}
		i--
		j--
	}
	return string(outputByte)
}
```
## 测试用例

```go
package q004

import "testing"

func TestAnswer(t *testing.T) {
	type args struct {
		inputStr string
	}
	tests := []struct {
		name          string
		args          args
		wantOutputStr string
	}{
		{
			"Function test 1",
			args{
				inputStr: "We Are Happy",
			},
			"We%20Are%20Happy",
		},
		{
			"Empty",
			args{
				inputStr: "",
			},
			"",
		},
		{
			"Just space",
			args{
				inputStr: " ",
			},
			"%20",
		},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if gotOutputStr := Answer(tt.args.inputStr); gotOutputStr != tt.wantOutputStr {
				t.Errorf("Answer() = %v, want %v", gotOutputStr, tt.wantOutputStr)
			}
		})
	}
}
```

[代码下载地址](https://github.com/leoryu/AGym/tree/master/sword/q004)