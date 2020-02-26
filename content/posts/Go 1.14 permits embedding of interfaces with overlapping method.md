---
title: "Go 1.14 permits embedding of interfaces with overlapping method"
date: 2020-02-26T15:37:34+08:00
draft: false
tags: ["1.14","interface"]
categories: ["go"]
---

在新发布的 Go 1.14 release <sup>[1](#reference-1)</sup> 中我们可一看到go team对代码进行了一系列的优化，其中发生了`Change to the language`的更新。在本文中作者将对这一更新进行尝试和探讨。

我们知道Go是不支持继承只支持组合的，在结构体中发生同名字段组合的时候，结构体本身字段的优先级是高于被组合结构体的，可以通过以下代码进行证明。

```go
package main

import "fmt"

// Animal animal.
type Animal struct {
	Name string
}

// Cat cat.
type Cat struct {
	Animal
	Name string
}

func main() {
	var a = Animal{Name: "animal"}
	var c = Cat{Animal: a, Name: "cat"}
	fmt.Println(c.Name)
}
// Output:
// cat
```

而对于接口(interface)，如果在组合的过程中出现函数签名重复的情况则在使用时会发生`ambiguous selector`，可以通过以下代码证明。

```go
package main

import "fmt"

// Animal Animal.
type Animal interface {
	Name() string
	Age() int
}

// Dog Dog.
type Dog interface {
	Name() string // custom Name func
	Animal
}

// Pug pug.
type Pug struct {
}

// Name Name.
func (pug *Pug) Name() string { return "pug" }

// Age Age.
func (pug *Pug) Age() int { return 1 }

func main() {
	var dog Dog
	dog = &Pug{}
	fmt.Println(dog.Name()) // go1.13 will cause ambiguous selector
	fmt.Println(dog.Age())
}
```

此次更新前，我们只能通过删除被组合类型(Animal)或组合类型(Dog)的方法来解决这个问题，这在实际开发过程中是非常不友好的，也会使得代码变得难以维护。
    
这次的更新使得我们在组合类型中定义方法会覆盖被组合类型的方法，以上代码在go1.14下可以正常编译运行。

### Reference:

<a id="reference-1"></a>1. https://golang.org/doc/go1.14#language "Go 1.14 release document"

<a id="reference-2"></a>2. https://github.com/golang/proposal/blob/master/design/6977-overlapping-interfaces.md "overlapping interfaces proposal"
