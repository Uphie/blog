---
title: Go 中 new 和 make 的区别
author: Uphie
date: 2022-12-14 12:01:20 +0800
categories: [技术]
tags: [go]
math: true
toc: true
---

Go 中 new 和 make 都是创建对象的一种方式，但它们有一些区别。

# new

我们先来看官方权威的解释：
```go
// Go 1.19.3
// src/builtin/builtin.go

// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.
func new(Type) *Type
```

以上说明中有几个重点：
- 函数功能是用来分配内存
- 参数是类型，而非值
- 返回值是指针
- 返回值指向的是新分配的指定类型的零值对象

使用示例：
```go
func main(){
    var i = new(int)
    // *int
    fmt.Println(reflect.TypeOf(i))
    *i = 10
    // 10
    fmt.Println(*i)

    var a = new(Animal)
    // *main.Animal
    fmt.Println(reflect.TypeOf(a))
    // &{Cat}
    a.Kind = "Cat"
    fmt.Println(a)
}
```

# make

现在我们看下关于 make 的解释：
```go
// Go 1.19.3
// src/builtin/builtin.go

// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
//
//	Slice: The size specifies the length. The capacity of the slice is
//	equal to its length. A second integer argument may be provided to
//	specify a different capacity; it must be no smaller than the
//	length. For example, make([]int, 0, 10) allocates an underlying array
//	of size 10 and returns a slice of length 0 and capacity 10 that is
//	backed by this underlying array.
//	Map: An empty map is allocated with enough space to hold the
//	specified number of elements. The size may be omitted, in which case
//	a small starting size is allocated.
//	Channel: The channel's buffer is initialized with the specified
//	buffer capacity. If zero, or the size is omitted, the channel is
//	unbuffered.
func make(t Type, size ...IntegerType) Type
```

重点是：
- 函数功能是用来为 `slice`、`map`或 `channel` 分配内存并初始化
- 首个参数为类型，剩余参数根据类型不同含义不同功能
- 返回值根据类型参数不同而不同，但都是初始化后了的对象

使用示例：
```go
type Animal struct {
	Kind string
}

func main() {
	var s1 = make([]Animal, 5)
    // s1,type:[]main.Animal,value:[{} {} {} {} {}],len:5,cap:5
	fmt.Printf("s1,type:%v,value:%v,len:%d,cap:%d\n", reflect.TypeOf(s1), s1, len(s1), cap(s1))

	var s2 = make([]int, 2, 6)
    // s2,type:[]int,value:[0 0],len:2,cap:6
	fmt.Printf("s2,type:%v,value:%+v,len:%d,cap:%d\n", reflect.TypeOf(s2), s2, len(s2), cap(s2))

	var m1 = make(map[int]string)
    // m1,type:map[int]string,value:map[],len:0
	fmt.Printf("m1,type:%v,value:%+v,len:%d\n", reflect.TypeOf(m1), m1, len(m1))

	var m2 = make(map[int]Animal, 3)
    // m2,type:map[int]main.Animal,value:map[],len:0
	fmt.Printf("m2,type:%v,value:%+v,len:%d\n", reflect.TypeOf(m2), m2, len(m2))

	var c1 = make(chan int)
    // c1,type:chan int,value:0xc00005c0c0,len:0,cap:0
	fmt.Printf("c1,type:%v,value:%+v,len:%d,cap:%d\n", reflect.TypeOf(c1), c1, len(c1), cap(c1))
	var c2 = make(chan Animal, 5)
    // c2,type:chan main.Animal,value:0xc000050180,len:0,cap:5
	fmt.Printf("c2,type:%v,value:%+v,len:%d,cap:%d\n", reflect.TypeOf(c2), c2, len(c2), cap(c2))
}
```

# 小结

相同点：都可以分配内存。

不同点：
new，用于任意类型的内存分配，参数为类型，返回创建的零值对象的指针。
make，只用于 slice、map、channel 的内存分配和初始化，参数为类型以及可选参数，返回对象。