---
Date: 2016-08-04T22:50:37.044674469+08:00
Title: go 语言学习笔记
date: 2016-08-04T22:50:37+08:00
description: 要么旅行，要么读书，身体和思想，必须有一个在路上
isCJKLanguage: "true"
tags:
- go
- development
- garbage collection
topics:
- go 语言学习笔记
url: "reading/qyuhenGo"
---

## 14章-15章
go 在初始化的过程中做了些什么，包括内存的初始化，gc 初始化，已经 main 函数和各个包 init 的初始化

## 16章 内存分配
go 自主管理内存分配，有个很重要的原因是为了配合垃圾回收
评价内存分配器的一个点：在性能和利用率之间做到平衡 

