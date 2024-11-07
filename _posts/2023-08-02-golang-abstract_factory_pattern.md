---
layout: post
title: "抽象工厂设计模式"
categories: golang
author:
- xihua
meta: "Springfield"
---

### 定义:
抽象工厂设计模式，是一种创造性设计模式, 可以创建一些列相关对象。是一种工厂设计模式的抽象。Let's say we have two factories.

- nike 
- Adidas

Imagine you need to buy a sports kit which has a shoe and short.Preferbly most of the time you would want to buy a full sports kit of a similar factory. i.e either nike or adidas.This is where the abstract factory pattern comes into the picture as concrete products that you want is shoe and short and these products will be created by the abstract factory of nike and adidas.

Both these two factories - nike and adidas implement iSportsFactory interface. We have two product interfaces.
    
- iShoe - this interface is implemented by nikeShoe and adidasShoe concrete products
- iShort - this interface is implemented by NikeShort and adidasShort concrete products.

Now let's look at code

### Code:

#### iSportsFactory.go
```
package main

import "fmt"

type iSportsFactory interface {
    makeShoe() iShoe
    makeShort() iShort
}

func getSportsFactory(brand string) iSportsFactory {
    if brand == "adidas" { 
        return &adidasFactory{}
    }
    if brand == "nike" {
        return &nikeFactory{}
    }
}
```
#### iShoe.go
```
package main

type iShoe interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type shoe struct {
    logo string
    size int
}

func (s *shoe) setLogo(logo string) {
    s.logo = logo 
}

func (s *shoe) setSize(size int) {
    s.size = size 
}

func (s *shoe) getLogo() string {
   return s.logo
}

func (s *shoe) getSize() int {
    return s.size
}

type adidasShoe struct {
    shoe
}

type nikeShoe struct {
   shoe 
}
```

#### iShort.go
```
package main

type iShort interface {
 setLogo(logo string)
 setSize(size int)
 getLogo() string
 getSize() int
}

type short struct {
    logo string
    size int
}

func (s *short) setLogo(logo string) {
    s.logo = logo
}

func (s *short) setSize(size int) {
    s.size = size
}

func (s *short) getLogo() string {
    return s.logo
}

func (s *short) getSize() int {
    return s.size
}

type adidasShort struct {
    short
}

type nikeShort struct {
    short
}
```

#### adidas_factory.go
```
package main

type adidasFactory struct {
}

func (a *adidasFactory) makeShoe() iShoe {
    return &adidasShoe{shoe{logo: "adidas", size: 14}}
}

func (a *adidasFactory) makeShort() iShort {
    return &adidasShort{short{logo: "adidas", size: 10}}    
}
```
#### nike_factory.go
```
package main

type nikeFactory struct {
}

func (n *nikeFactory) makeShoe() iShoe {
    return &nikeShoe{shoe{logo: "nike", size: 14}}    
}

func (n *nikeFactory) makeShort() iShort {
    return &nikeShort{short{logo: "nike", size: 10}}
}
```
#### main.go
```
package main

func main() {
    adidasFactory := getSportsFactory("adidas")
    nikeFactory := getSportsFactory("nike")

    adidasShoe := adidasFactory.makeShoe()
    adidasShort := adidasFactory.makeShort()

    nikeShoe := nikeFactory.makeShoe()
    nikeShort := nikeFactory.makeShort()

    fmt.Println(adidasShoe.getLogo(), adidasShoe.getSize())
    fmt.Println(adidasShort.getLogo(), adidasShort.getSize())
    fmt.Println(nikeShoe.getLogo(), nikeShoe.getSize())
    fmt.Println(nikeShort.getLogo(), nikeShort.getSize())
}
```

