---
title: fatal error, all goroutines are asleep - deadlock!
author: Uphie
date: 2022-02-16 22:20:20 +0800
categories: [技术]
tags: [go,goroutine]
math: true
toc: true
---


我们先看下下面这段代码：
```go
package main

import (
	"fmt"
	"time"
)

func write(c chan int) {
	for i := 0; i < 10; i++ {
		c <- 1
		time.Sleep(time.Second)
	}
}

func main() {
	var c = make(chan int)

	go write(c)

	for e := range c {
		fmt.Println("from channel get ", e)
	}
}
```

程序执行结果：
```console
from channel get  1
from channel get  1
from channel get  1
from channel get  1
from channel get  1
from channel get  1
from channel get  1
from channel get  1
from channel get  1
from channel get  1
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.main()
        channel.go:20 +0xe5
exit status 2
```

goroutine 死锁了，因为 `all goroutines are asleep`。为什么会 asleep 呢？


我们仔细看看上面的代码会发现，`write` 向 channel 写完数据后，又从 channel 遍历读取完了数据。
然后读操作会阻塞等待channel 写操作，即 asleep；而写操作在等待channel读操作，也在阻塞，也即 asleep。
双方都等待对方执行，于是就死锁了。

要解决这个问题，需要从写的这一方解决，写完数据后关闭 channel：
```go
func write(c chan int) {
	for i := 0; i < 10; i++ {
		c <- 1
		time.Sleep(time.Second)
	}
	close(c)
}
```

对于未关闭的 channel，for 循环读取不到数据后就会阻塞，直到写操作执行；但是对于已关闭的 channel，for 循环读取不到数据后就会结束退出循环。
