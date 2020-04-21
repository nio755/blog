---
title: GoLang 逃逸分析
date: 2020-04-19T20:44:45+08:00
lastmod: 2020-04-19T20:44:45+08:00
author: nio
cover: /img/go-cover.jpg
categories: ["GoLang"]
tags: ["GoLang"]
# showcase: true
draft: true
---

- [介绍](#%e4%bb%8b%e7%bb%8d)
- [逃逸场景](#%e9%80%83%e9%80%b8%e5%9c%ba%e6%99%af)
  - [指针逃逸](#%e6%8c%87%e9%92%88%e9%80%83%e9%80%b8)
  - [栈空间不足逃逸（空间开辟过大）](#%e6%a0%88%e7%a9%ba%e9%97%b4%e4%b8%8d%e8%b6%b3%e9%80%83%e9%80%b8%e7%a9%ba%e9%97%b4%e5%bc%80%e8%be%9f%e8%bf%87%e5%a4%a7)
  - [动态类型逃逸（不确定长度大小）](#%e5%8a%a8%e6%80%81%e7%b1%bb%e5%9e%8b%e9%80%83%e9%80%b8%e4%b8%8d%e7%a1%ae%e5%ae%9a%e9%95%bf%e5%ba%a6%e5%a4%a7%e5%b0%8f)
  - [闭包引用对象逃逸](#%e9%97%ad%e5%8c%85%e5%bc%95%e7%94%a8%e5%af%b9%e8%b1%a1%e9%80%83%e9%80%b8)
- [逃逸分析的作用](#%e9%80%83%e9%80%b8%e5%88%86%e6%9e%90%e7%9a%84%e4%bd%9c%e7%94%a8)
- [逃逸总结](#%e9%80%83%e9%80%b8%e6%80%bb%e7%bb%93)
- [函数传递指针真的比传值效率高吗](#%e5%87%bd%e6%95%b0%e4%bc%a0%e9%80%92%e6%8c%87%e9%92%88%e7%9c%9f%e7%9a%84%e6%af%94%e4%bc%a0%e5%80%bc%e6%95%88%e7%8e%87%e9%ab%98%e5%90%97)
- [参考](#%e5%8f%82%e8%80%83)

## 介绍

**逃逸分析是编译器用来决定你的程序中值的位置的过程**。**特别地，编译器执行静态代码分析，以确定一个构造体的实例化值是否会逃逸到堆**。在 Go 语言中，你没有可用的关键字或者函数，能够直接让编译器做这个决定。只能够通过你写代码的方式来作出这个决定。

## 逃逸场景

### 指针逃逸

Go 可以返回局部变量指针，这其实是一个典型的变量逃逸案例，示例代码如下：

```go
package main

type Student struct {
    Name string
    Age  int
}

func StudentRegister(name string, age int) *Student {
    s := new(Student) //局部变量s逃逸到堆

    s.Name = name
    s.Age = age

    return s
}

func main() {
    StudentRegister("Jim", 18)
}
```

### 栈空间不足逃逸（空间开辟过大）

```go
package main

func Slice() {
    s := make([]int, 10000, 10000)

    for index, _ := range s {
        s[index] = index
    }
}

func main() {
    Slice()
}
```

当栈空间不足以存放当前对象时或无法判断当前切片长度时会将对象分配到堆中。

### 动态类型逃逸（不确定长度大小）

很多函数参数为 interface 类型，比如 fmt.Println(a …interface{})，编译期间很难确定其参数的具体类型，也能产生逃逸。

```go
package main

import "fmt"

func main() {
    s := "Escape"
    fmt.Println(s)
}
```

### 闭包引用对象逃逸

```go
package main

import "fmt"

func Fibonacci() func() int {
    a, b := 0, 1
    return func() int {
        a, b = b, a+b
        return a
    }
}

func main() {
    f := Fibonacci()

    for i := 0; i < 10; i++ {
        fmt.Printf("Fibonacci: %d\n", f())
    }
}
```

Fibonacci()函数中原本属于局部变量的 a 和 b 由于闭包的引用，不得不将二者放到堆上，以致产生逃逸。

## 逃逸分析的作用

1. 逃逸分析的好处是为了**减少gc的压力**，不逃逸的对象分配在栈上，当函数返回时就回收了资源，不需要gc标记清除。
2. 逃逸分析完后可以确定哪些变量可以分配在栈上，**栈的分配比堆快，性能好**(逃逸的局部变量会在堆上分配 ,而没有发生逃逸的则有编译器在栈上分配)。
3. **同步消除**，如果你定义的对象的方法上有同步锁，但在运行时，却只有一个线程在访问，此时逃逸分析后的机器码，会去掉同步锁运行。

## 逃逸总结

- 栈上分配内存比在堆中分配内存有更高的效率
- 栈上分配的内存不需要GC处理
- 堆上分配的内存使用完毕会交给GC处理
- 逃逸分析目的是决定内分配地址是栈还是堆
- 逃逸分析在编译阶段完成

## 函数传递指针真的比传值效率高吗

我们知道传递指针可以减少底层值的拷贝，可以提高效率，但是如果拷贝的数据量小，由于指针传递会产生逃逸，可能会使用堆，也可能会增加GC的负担，所以传递指针不一定是高效的。

## 参考

- [Golang内存分配逃逸分析](https://driverzhang.github.io/post/golang%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90/)
