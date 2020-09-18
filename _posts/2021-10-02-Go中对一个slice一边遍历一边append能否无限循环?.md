---
title: Go中对一个slice一边遍历一边append能否无限循环下去？
author: Uphie
date: 2021-10-02 23:00:00 +0800
categories: [技术]
tags: [go,slice]
math: true
toc: true
---


如题，思考下面的代码，一边遍历， 一边append新元素，能否一直循环下去？
```go
package main

import "fmt"

func main() {
	s := []uint64{1, 2, 3}
	for _, i := range s {
		fmt.Println("got ", i)
		s = append(s, i)
	}
}
```
小白按常规的思维去想，一定会无限循环下去的，但到底是是真是假？

我们那拿 Python 跑一下;
```python
s = [1, 2, 3]

for i in s:
    print(f"got {i}")
    s.append(i)
```

结果：
```
got 1
got 2
got 3
got 1
got 2
got 3
got 1
...
```
不出所料，无限循环下去了。

那么 Go 呢？

跑一下：
```
got  1
got  2
got  3
```

我们发现结果并不是那样，那么为什么呢？

对于所有的for range 循环，Go在编译期间会将遍历对象（数组、切片） 赋值给一个临时变量，取遍历对象的长度，对于不同的遍历情况进行不同的优化处理。

对于 `for idx,val:=range data` 这种遍历取索引和值的情况，编译器会编译成如下形式处理：
```go
// a 为原对象
ha := a
// 遍历的索引，初始化为0
hv1 := 0
// 遍历最大次数
hn := len(ha)
// 遍历到的索引
v1 := hv1
// 遍历到的值
v2 := nil
for ; hv1 < hn; hv1++ {
    tmp := ha[hv1]
    v1, v2 = hv1, tmp
    ...
}
```
对于 `for idx:=range data` 这种只遍历取索引的情况，编译器会编译成如下形式处理：
```go
ha := a
// 遍历初始索引
hv1 := 0
hn := len(ha)
v1 := hv1
for ; hv1 < hn; hv1++ {
    v1 = hv1
    ...
}
```

对于 `for range data` 这种只遍历不取值的情况，编译器会编译成如下形式处理：
```go
ha := a
// 遍历初始索引
hv1 := 0
hn := len(ha)
v1 := hv1
for ; hv1 < hn; hv1++ {
    ...
}
```

前面我们提到的问题中，由于遍历时遍历的次数已经被固定了，所以无法无限循环迭代，只会循环一次。

参考资料：https://github.com/golang/go/blob/41d8e61a6b9d8f9db912626eb2bbc535e929fefc/src/cmd/compile/internal/gc/range.go#L216