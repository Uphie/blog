---
title: 为什么父 goroutine 不能恢复子 goroutine 的 panic？
author: Uphie
date: 2023-02-01 21:13:00 +0800
categories: [技术]
tags: [go,goroutine]
math: true
toc: true
---

先回忆下几个 panic 情形：
```go
// 主动 panic
// panic: bad condition
panic("bad condition")


// 下标越界
// panic: runtime error: index out of range [1] with length 0
m := make([]int, 0)
fmt.Println(m[1])

// 关闭 nil 的 channel
// panic: close of nil channel
var ch chan int
close(ch)

// 断言失败
// panic: interface conversion: interface {} is int, not string
var v interface{} = 10
fmt.Println(v.(string))
```

如题，为什么父 goroutine 不能恢复子 goroutine 的 panic？换句话说更普遍的问题是为什么不能 recover 其他 goroutine 里产生的 panic？

这个问题是一个有争议的问题，讨论过很久，问题的原因可以简单地认为是设计问题：**goroutine 被设计为一个独立的代码执行单元，拥有自己的执行栈，不与其它 goroutine 共享任何数据。这意味着，不能让 goroutine 拥有返回值，也无法让 goroutine 拥有自己的 ID 编号等**。

如何解决呢？要解决的话必然涉及到通信问题，可以将 `recover()` 得到的错误信息通过 channel 发送到全局的的捕获中心，记录下来。

简易代码示例如下：
```go
package main

import (
	"fmt"
	"time"
)

var notifier chan interface{}

// 启动通知监听，收到通知就记录下来
func startPanicRecording() {
	notifier = make(chan interface{})
	go func() {
		for {
			select {
			case r := <-notifier:
				// 记录下来
				fmt.Println("got panic:", r)
			}
		}
	}()
}

// 对 goroutine 进行包装，增加 recover 
func Go(f func()) {
	go func() {
		defer func() {
			if r := recover(); r != nil {
				notifier <- r
			}
		}()
		f()
	}()
}
func main() {
	startPanicRecording()

	Go(func() {
		m := make([]int, 0)
        // 下标越界，产生 panic
		fmt.Println(m[0])
	})
    // 避免主 goroutine 过快结束
	time.Sleep(time.Second)
}
```

这个方案并不完美，如果 `GO(f func())` 的参数没有嵌套调用 `GO(f func())` 仍然阻止不了 panic，而且对于下面这样的代码也恢复不了： 

```go
func main() {
	m := make(map[int]int)
	var wg sync.WaitGroup
    // 并发写 map，造成 fatal error
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(data int) {
			m[data] = data * data
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

在 Go 官方没有提供特别完美的方案之前，建议开发者写好稳健的代码，做好充分的测试，再在运维上增加容灾机制。