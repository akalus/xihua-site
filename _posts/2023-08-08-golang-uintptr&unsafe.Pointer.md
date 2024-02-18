---
layout: post
title: "Understanding uintptr in Golang"
categories: golang
author:
- xihua
meta: "Springfield"
---

This is an unsingned integer type which is large enough to hold any pointer address.Therefore its size is platform dependent.It is just an integer representation of an address.

## Properties

* A uintptr can be converted to unsafe.Pointer and viceversa. Later we will talk about where conversion of uintptr to unsafe.Pointer is useful.(uintptr 和 unsafe.Pointer 可以互相转换)
* Arithmetic can be performed on the uintptr. Do note here arithmetic cannot be performed in a pointer in Go or unsafe.Pointer in Go. 
* uintptr even though it holds a pointer address, is just a value and does not reference any object. Therefore
    * Its value will not be updated if the corresponding object moves.Eg When goroutine stack changes, uintptr value will not change.
    * The corresponding object can be garbage collected.The GC does not consider uintptr as live reference and hence the can be garbage collected.

## Purpose

uintptr can be used for below purposes:

* One purpose of uintptr is to be used along
