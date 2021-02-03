---
title: golang 中 slice 和 array 的区别
date: 2021-02-01 21:38:00
tags: [golang, array, slice]
cover: https://cdn.jsdelivr.net/gh/bob-zou/bob-zou.github.io/source/covers/go-array-vs-slice.png
---

## 概要
| Array | Slice |
| --- | --- |
| 底层是一段连续的内存空间 | 底层是一个指向数组的指针(数组指针) |
| 只有len属性, 且长度固定 | 有len和cap属性, 且不固定 |
| Go数组是值类型, 赋值和函数传参操作都会复制整个数组数据 | 引用类型 |

## 扩展
### 大数组传参
一个大数组传递给函数会消耗很多内存，采用切片的方式传参可以避免上述问题。
切片是引用传递，所以它们不需要使用额外的内存并且比使用数组更有效率。

### 性能对比
**测试代码：**
```go
package main

import "testing"

const SIZE = 100

func array() [SIZE]int {
	var x [SIZE]int
	for i := 0; i < len(x); i++ {
		x[i] = i
	}
	return x
}

func slice() []int {
	x := make([]int, SIZE)
	for i := 0; i < len(x); i++ {
		x[i] = i
	}
	return x
}

func BenchmarkArray(b *testing.B) {
	for i := 0; i < b.N; i++ {
		array()
	}
}

func BenchmarkSlice(b *testing.B) {
	for i := 0; i < b.N; i++ {
		slice()
	}
}
```
**执行结果：**
> 各个字段含义: 核数(4)，循环次数，平均每次执行时间，每次执行堆上分配内存总量，分配次数
```shell
SIZE=100
BenchmarkArray-4         3633415               312 ns/op               0 B/op          0 allocs/op
BenchmarkSlice-4         2575620               499 ns/op             896 B/op          1 allocs/op
```
```shell
SIZE=1000
BenchmarkArray-4          437178              2835 ns/op               0 B/op          0 allocs/op
BenchmarkSlice-4          304779              3781 ns/op            8192 B/op          1 allocs/op
```
```shell
SIZE=10000
BenchmarkArray-4           38402             27179 ns/op               0 B/op          0 allocs/op
BenchmarkSlice-4           33938             36109 ns/op           81920 B/op          1 allocs/op
```
```shell
SIZE=100000
BenchmarkArray-4            3666            294832 ns/op               0 B/op          0 allocs/op
BenchmarkSlice-4            2868            358106 ns/op          802818 B/op          1 allocs/op
```
```shell
SIZE=1000000
BenchmarkArray-4             213           5484479 ns/op               0 B/op          0 allocs/op
BenchmarkSlice-4             358           3994363 ns/op         8003590 B/op          1 allocs/op
```
```shell
SIZE=10000000
BenchmarkArray-4               7         170413247 ns/op        80003074 B/op          1 allocs/op
BenchmarkSlice-4              42          27718152 ns/op        80003080 B/op          1 allocs/op
```
这样对比看来，并非所有时候都适合用切片代替数组，因为切片底层数组可能会在堆上分配内存，而且小数组在栈上拷贝的消耗也未必比 make 消耗大。

### Array 修改拷贝的值不影响原来的数组
```go
package main

import "fmt"

func main() {
    a := [5]int{1, 2, 3, 5, 6}
    b := a
    b[0] = 10
    fmt.Println(a, b) // [1 2 3 5 6] [10 2 3 5 6]
}
```

### Slice 底层是数组指针, 修改拷贝的值影响原来的切片
```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 5, 6}
	b := a
	b[0] = 10
	fmt.Println(a, b) // [10 2 3 5 6] [10 2 3 5 6]
}
```

### 在cap不足时, append会返回新的切片
> 当原来切片的cap小于1024时, 新切片的cap为原来的 2 倍
> 当原来切片的cap大于等于1024时, 新切片的cap为原来的 1.25 倍
```go
package main

import "fmt"

func main() {
	a := []int{1, 2, 3, 5, 6}
	b := a
	b = append(b, 7)
	b[0] = 10
	fmt.Println(a, len(a), cap(a)) // [1 2 3 5 6] 5 5
	fmt.Println(b, len(b), cap(b)) // [10 2 3 5 6 7] 6 10
}
```
