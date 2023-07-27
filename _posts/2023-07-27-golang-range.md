---
layout: post
title: "range loop in go"
categories: golang
author:
- xihua
meta: "Springfield"
---

When it comes to loop, golang has:

* for loop
* for-range loop

for-range 循环用于迭代不同的数据集合，比如：

* array、slice
* string
* maps
* channel

### for-range 便利数组或切片

```golang
    for index, value := range array/slice {
        // Do something with index and value
    }
```

注意的是 index 从0 开始，value 就是当前循环到的值

### for-range 便利字符串

在golang 中 string 是一串bytes。 一个字符串本质上是一个连续的utf-8编码的字符。如果是前128个字符，占用一个字节，其他的占用1-4个字节。

```golang
sample := "a£c"
```

对于上面这个字符串

* 'a' 占用一个字节
* '£' 占用两个字节
* 'b' 占用一个字节

所以上面的字符一共占用 1 + 2 + 1 = 4 个字节。当打印这个字符串长度时，输出的是4而不是三。

```golang
    fmt.Printf("Length is %d\n", len(sample))
```

所以不能单个字节的来遍历字符串。下面这个打印，输出的是单个字节的值，而不是字符的值。

```golang
 for i := 0; i < len(sample); i++ {
    fmt.Printf("%c\n", sample[i])
 }
```

Output:

```golang
aÂ£b
```

大多数情况，上面的输出不是我们想要的。这是因为其输出的是单个字节的值。下面是使用for-range 来遍历字符串

```golang
for index, character := range string {
    //Do something with index and character
}
```

在开始上面这段代码之前有几个重点

* index 是 Unicode 编码的字符的在string 中的位置，'a' 的起始是0，'£'的起始位置是1，'b'的起始位置是3。
* 便利的值是Unicode 编码的字符，而不是字节。
* index 和 value 都是可选的。

```golang
package main

import "fmt"

func main() {
    sample := "a£b"

    //With index and value
    fmt.Println("Both Index and Value")
    for i, letter := range sample {
        fmt.Printf("Start Index: %d Value:%s\n", i, string(letter))
    }

    //Only value
    fmt.Println("\nOnly value")
    for _, letter := range sample {
        fmt.Printf("Value:%s\n", string(letter))
    }

    //Only index
    fmt.Println("\nOnly Index")
    for i := range sample {
        fmt.Printf("Start Index: %d\n", i)
    }
}
```

Output:

```golang
Both Index and Value
Start Index: 0 Value:a
Start Index: 1 Value:£
Start Index: 3 Value:b

Only value
Value:a
Value:£
Value:b

Only Index
Start Index: 0
Start Index: 1
Start Index: 3
```

### for-range 循环 channel

for-range loop works differently too for a channel. For a channel, an index doesn't make any sense as the channel is similar to a pipeline where values enter from one and exit from the other end.

So in case of channel, the for-range loop will iterate over values currently present in the channel. After it has iterated over all the values currently present. the for-range loop will not exit but instead wait for next value that might be pushed to the channel and it will exit only when the channel is closed

channl 使用 for-range 的方式如下：

```golang
for value := range channel {
    //Do something value
}
```

Let's see a code example

```golang
package main

import "fmt"

func main() {
    ch := make(chan string)
    go pushToChannel(ch)
    for val := range ch {
        fmt.Println(val)
    }
}
func pushToChannel(ch chan<- string) {
    ch <- "a"
    ch <- "b"
    ch <- "c"
    close(ch)
}
```

Output:

```golang
a
b
c
```
