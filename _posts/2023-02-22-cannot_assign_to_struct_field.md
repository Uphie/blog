---
title: cannot assign to struct field m["bob"].Name in map
author: Uphie
date: 2023-02-22 18:43:00 +0800
categories: [技术]
tags: [go]
math: true
toc: true
---

观察下下面代码会怎样执行？
```go
package main

import "fmt"

type User struct {
	Name string
}

func main() {
	m := map[string]User{"bob": {Name: "bob"}}
	m["bob"].Name = "andy"
	fmt.Sprintln("m:", m)
}
```
结果是编译错误：
```
cannot assign to struct field m["bob"].Name in map
```

原因是 `m["bob"]` 是**不可寻址**的。 可直接使用 `&` 操作符取地址的对象，就是可寻址的（Addressable）。

不可寻址的有：

 1. 常量

 ```go
 const SIZE = 100

// invalid operation: cannot take address of SIZE (untyped int constant 100)
fmt.Println(&SIZE)
 ```

 2. 字符串
 ```go
 // invalid operation: cannot take address of "hello" (untyped string constant)
fmt.Println(&"hello")
 ```

 3. 函数或方法
 ```go
func sum(x, y int) int {
	return x + y
}

// invalid operation: cannot take address of sum (value of type func(x int, y int) int)
fmt.Println(&sum)

type User struct {
	Name string
}

func (u *User) Walk() {
	fmt.Printf("%s is walking\n", u.Name)
}
// invalid operation: cannot take address of u.Walk (value of type func())
fmt.Println(&u.Walk)
 ```

 4. 字面量

基础字面量：
 ```go
// invalid operation: cannot take address of 1 (untyped int constant)
fmt.Println(&1)
// invalid operation: cannot take address of 1.1 (untyped float constant)
fmt.Println(&1.1)
 ```

组合字面量：
 ```go
 type User struct {
	Name string
}

func (u *User) Walk() {
	fmt.Printf("%s is walking\n", u.Name)
}

func NewUser(name string) User {
	return User{Name: name}
}
// invalid operation: cannot take address of NewUser("uphie") (value of type User)
fmt.Println(&NewUser("uphie"))
```

5. map中的元素

可以拿文中开头的错误作为例子。为什么map中的元素不可寻址？

如果可寻址，
1. 如果map中元素不存在，则返回零值，但零值是不可变对象
2. 如果map中元素存在，map中元素可能因为扩容、缩容地址发生变化，那么取到的地址将是无意义的

6. 数组字面量

```go
// invalid operation: [3]int{…} (value of type [3]int) (slice of unaddressable value)
fmt.Println([3]int{1, 2, 3}[1:])
```
