---
layout: post
title: "Golang中的闭包"
categories: golang
author:
- xihua
meta: "Springfield"
---

golang中的闭包，是函数和其引用环境的结合。在Go底层是通过一个结构体来实现的。

type closure struct {
    f func()
    args []interface{}
}

golang 返回一个闭包其实返回的是一个结构体，这个结构体中包含了方法和需要的参数。通过escape analyze 来确定变量是分配到堆还是栈。
在golang 闭包中使用外部变量的时候，一直是引用传递。
