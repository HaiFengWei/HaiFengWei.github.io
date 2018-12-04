---
title: 初识Swift
date: 2018-10-06 22:21:51
tags:
- Swift
- 语法
categories: Swift
---

**拥抱Swift,迎接未来**


如今Swift4.0已经发布，各方面已经趋于成熟。相对于OC其简洁的语法、精准报错、定义变量简单等种种迹象表明着是苹果的新宠，而OC则成为过去时<!--more-->

# 基础语法阅读
### 可变变量与不可变变量
*  可变变量修饰符`var`,若不指定类型，编译器可以根据给变量所赋的值，推断它的数据类型

```objc
\\ String类型
var name = "xiaoming"
var tempName : String = "tempXiaoming"
name = "变量可改变"
\\ Int类型
var age  = 1 
var tempAge : Int  = 1 
```
*  不可变变量修饰符`let`,若不指定类型，编译器可以根据给常量所赋的值，推断它的数据类型

```objc
\\ String类型
let name = "xiaoming"
let tempName : String = "tempXiaoming"
\\ name = "不可变变量不可改变" 
\\ Int类型
let age  = 1 
let tempAge : Int  = 1 
```

