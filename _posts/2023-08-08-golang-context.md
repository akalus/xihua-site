---
layout: post
title: "Golang中的常量"
categories: golang
author:
- xihua
meta: "Springfield"
---

## Definition

Context is a package provided by Go. Let's first understand some problems that existed already, and which context package tries to sovle.

## Problem Statement

* Let's say that you started a function and you need to pass some common parameters to the downstream functions. You cannot pass thess common parameters each as an argument to all the downstream functions.
* You started a goroutine which in turn start more goroutines and so on. Suppose the task that you were doing is no longer needed. Then how to inform all child goroutines go gracefully exit or return.
* A task should be finished within a specified timeout of say 2 seconds.If not it should gracefully exit or return.
* A task should be finished within a deadline eg it should end before 5 pm. If not finished then is should gracefully exit and return.

if you notice all the above problems are quite applicable to HTTP requests and but none the less these problems are also applicable to many different areas too.

For a web HTTP request, it needs to be canceled when the client has disconnected, or the request has to be finished within a specified timeout and also requests scope values such as request_id needs to be available to all downstream functions.

## When to Use(Some Use cases)

* To pass data to the downstream.Eg. a HTTP request creates a request_id, request_user which needs to be passed around to all downstream functions for distributed tracing.
* When you want to halt the operation within the midway - A HTTP request should be stopped because the client disconnected.
* When you want to halt the operation within a specified time from start i.e with timeout -Eg- a HTTP request should be completed in 2 sec or else should be aborted.
* When you want to halt an operation before a certain time -Eg. A cron is running that needs to be aborted in 5 mins of not completed.

## Context Interface

The core of understanding context if knowing the Context interface

```golang
type Context interface {
    //It retures a channel when a context is cancelled, timesout (either when deadline is reached or timeout time has finished)
    Done() <-chan struct{}

    //Err will tell why this context was cancelled. A context is cancelled in three scenarios.
    // 1. With explicit cancellation signal
    // 2. Timeout is reached
    // 3. Deadline is reached
    Err() error

    //Used for handling deallines and timeouts
    Deadline() (deadline time.Time, ok bool)

    //Used for passing request scope values
    Value(key interface{}) interface{}
}
```

## Creating New Context

### context.Background():

context package function Background() returns a empty Context which implements the context interface

    1. It has no values
    2. It is never cannceled
    3. It has no deadline

 Then what is the use context.Background(). context.Background() serves as the root of all context which will be derived from it. It will be more clear as we go along.

### context.ToDo():

* context package ToDo function returns as empty Context. This context is used when the surrounding function has not been passed a context and one wants to use the context as a placeholder in the current function and plans to add actual context in the near future. One use of adding it as a placeholder is that it helps in validation in the Static Code Analysis tool.

The above two methods describe a way of creating new contexts. More context can be derived from thess contexts. This is where context tree comes into the picture.

## Context Tree

Before understanding Context Tree please make soure that it is implicitly created in the background when using context. You will find no memtion of in go context package itself.

Whenever you use context, then the emtpy Context got from context.Background() is the root of all context. Context.ToDo() also acts like root context but as mentioned above it is more like a context placeholder for future use. This empty context has no functionality at all and we can add functionality by deriving a new context from this. Basically a new context is created by wrapping an already existing immutable context and adding additional infomation. Let's see more examples of a context tree which gets created

### Two Level tree

```golang
    rootCtx := context.Background()
    childCtx := context.WithValue(rootCtx, "msgId", "someMgId")
```

In above

* rootCtx is the empty Context with no functionality
* childCtx is derived from rootCtx and has the functionality of storing request-scoped values. In above example it is storing key-value pair of {"msgId":"someMsgId"}

### Three level tree 

```golang
    rootCtx := context.Backgroun()
    childCtx := context.WithValue(rootCtx, "msgId", "someMsgId")
    childOfChildCtx, cancelFunc := context.WithCancel(childCtx)
```

In above

* rootCtx is the empty context with no functionality
* childCtx is derived from rootCtx and has the functionality of storing request-scoped values. In above example it is storing key-value pair of {"msgId":"someMsgId"}
* childOfChildCtx is derived from childCtx. It has the functionality of storing request-scoped values and also it has the functionality of triggering cancellation signals. cancelFunc can be used to trigger cancellation signals

## Deriving From Context

A derived context is can be created in 4 ways

* Passing request-scoped values - using WithValue() function of context package
* With cancellation signals - using WithCancel() function of context package
* With deadlines - using WithTimeout() function of context package