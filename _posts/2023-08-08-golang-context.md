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
* With deadlines - using WithDeadline() function of context package
* With timeouts - using WithTimeout() function of context package

Let's understand each of the above in details

Used for passing request-scoped values. The complete signature of the function is

```golang
    withValue(parent Context, key, value interface{})(ctx Context)
```

It takes in a parent context,key,value and returns a derived context The derived context has key associated with the value.Here the parent context can be either context.Background() or any other context.Further,any context which is derived from this context will have this value.

```golang
    ctxRoot := context.Background()
    ctxChild := context.WithValue(ctxRoot, "a", "x")
    #Below ctxChildofChild has access to both pairs {"a":"x", "b":"y"} as it is derived from ctxChild
    ctxChildofChild  := context.WithValue(ctxChild, "b", "y")
```

* injectMsgID is a net http middleware function that populates the "msgID" field in context
* HelloWorld is the handler function for api "localhost:8080/welcome" which gets this msgID from context and sends it back as response headers

```golang
package main

import (
    "context"
    "net/http"
    "github.com/google/uuid"
)

func main() {
    hellowordHandler := http.HandlerFunc(helloword)
    http.Handle("/welcome", injectMsgID(helloWorldHandler))
    http.ListenAndServe(":8080", nil)
}

func HelloWorld(w http.ResponseWriter, r *http.Request) {
    msgID := ""
    if m := r.Context().Value("msgID");m != nil {
        if value, ok := m.(string);ok {
            msgID = value
        }
    }
    w.Header().Add("msgID", msgID)
    w.Write([]byte("hello, world"))
}
func injectMsgID(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        msgID := uuid.New().String()
        ctx := context.WithValue(r.Context(), "msgID", msgID)
        req := r.WithContext(ctx)
        next.ServeHTTP(w, req)
})
}
```

Simply do a curl call to the above request after running the above program

```golang
    curl -v http://localhost/welcome
```

Here will be the response.Notice the MsgID that gets populated in the response headers. The injectMsgID function acts as middleware and injects as unique msgID to the request context.

```golang
curl -v http://localhost:8080/welcome
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /do HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Msgid: a03ff1d4-1464-42e5-a0a8-743c5af29837
< Date: Mon, 23 Dec 2019 16:51:01 GMT
< Content-Length: 12
< Content-Type: text/plain; charset=utf-8
< 
* Connection #0 to host localhost left intact
```

### context.WithCancel()

Used for cancellation signals.Below is the signature of WithCancel() function

```golang
    func WithCancel(parent Context)(ctx Context, cancel CancelFunc)
```

context.WithCancel() function returns two things

* Copy the parentContext with the new done channel.
* A cancel function which when called closes this done channel

Only the creater of this context should call the cancel function. It is highly not re
