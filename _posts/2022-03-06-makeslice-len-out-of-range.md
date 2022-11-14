---
title: 创建 slice 出错
author: Uphie
date: 2022-03-06 13:18:00 +0800
categories: [技术]
tags: [go,slice]
math: true
toc: true
---

创建 slice 时出错：
```
panic: runtime error: makeslice: len out of range
```

原代码：
```go
func t() {
	length := 1 << 63 / 8
	s := make([]uint64, length, length)
	fmt.Println(s)
}
```

函数 `make([]uint64, length, length)` 在运行时调用的其实是
```go
// src/runtime/slice.go

func makeslice64(et *_type, len64, cap64 int64) unsafe.Pointer {
	len := int(len64)
	if int64(len) != len64 {
        // 操作系统非64位，len64超出 int所代表的位数时，会导致int64(len) != len64
		panicmakeslicelen()
	}

	cap := int(cap64)
	if int64(cap) != cap64 {
        // 操作系统非64位，cap64超出 int所代表的位数时，会导致int64(cap) != cap64
		panicmakeslicecap()
	}

	return makeslice(et, len, cap)
}


func makeslice(et *_type, len, cap int) unsafe.Pointer {
    // 计算所需内存，以及检测是否会溢出
	mem, overflow := math.MulUintptr(et.size, uintptr(cap))
	if overflow || mem > maxAlloc || len < 0 || len > cap {
		// NOTE: Produce a 'len out of range' error instead of a
		// 'cap out of range' error when someone does make([]T, bignumber).
		// 'cap out of range' is true too, but since the cap is only being
		// supplied implicitly, saying len is clearer.
		// See golang.org/issue/4085.
		mem, overflow := math.MulUintptr(et.size, uintptr(len))
		if overflow || mem > maxAlloc || len < 0 {
			panicmakeslicelen()
		}
		panicmakeslicecap()
	}

	return mallocgc(mem, et, true)
}
```
可以看到我们创建 slice 传的 length太大了，`length=1<<60=1152921504606846976`，要分配的内存超出了 `maxAlloc`，`maxAlloc` 是个常量，在不同平台大小不同。

有个 golang 的 [issue](https://github.com/golang/go/issues/38673) 也说明了发生这种错误的场景，结合源码我们可以看到当要分配的底层数组的内存大小溢出，或超出最大内存限制(`maxAlloc`)，或传入参数的长度为负数时，都会触发这个错误。


