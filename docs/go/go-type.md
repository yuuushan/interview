# Go 类型相关

## 基础

### ✨ Go 中 new 和 make 的区别

|                | **new**                                                        | **make**                 |
| -------------- | -------------------------------------------------------------- | ------------------------ |
| **作用**       | 计算类型大小,为其分配[零值](https://go.dev/tour/basics/12)内存 | 分配内存和初始化成员结构 |
| **参数类型**   | 任一类型                                                       | slice，map，channel      |
| **返回值类型** | 指针                                                           | 类型本身                 |

### Go 有没有引用类型？

Go 引用类型术语已经在 2013 年 4 月[从 Go specification 中删除](https://github.com/golang/go/commit/b34f0551387fcf043d65cd7d96a0214956578f94)，并说明 `spec: Go has no 'reference types'`。但引用类型的说法还是在社区广泛应用。

参考：[About the terminology "reference type" in Go](https://github.com/go101/go101/wiki/About-the-terminology-%22reference-type%22-in-Go)

### Go 有没有引用传递？

Go 只有值传递，没有引用传递。

## Slice

### Go slice 结构

Slice 有三个属性：

1. `array`: 指向数组的指针
2. `len`: 长度
3. `cap`: 容量

### 数组和切片有哪些区别？

1. 数组长度是固定的，使用前必须确定长度。切片长度可变。
2. 数组是基于连续的内存空间存储数据的，作为函数参数传递时传递的是数组的拷贝。切片底层是一个指向数组的指针。
3. 数组可以使用 `==` 和 `!=` 来比较；切片只能和 `nil` 比较。

### Slice 作为参数时，函数内部对 slice 扩容后函数外部的 slice 是否改变？

分两种情况讨论。

一是扩容后 slice 的 cap 没有改变。此时函数外部 slice 和函数内部 slice 指向的底层数组一致，但是 `len` 没有改变，因此看不到 `append` 扩容的内容，但对扩容前长度内的值修改会影响到函数外。

另一种情况是扩容后 slice 的 cap 发生了改变。此时 `append` 操作会重新创建一个新的底层数组，并将旧数组的值复制进去，然后回收旧数组。这种情况下函数内外的 slice 就完全不一样了，对函数内 slice 的修改不会影响函数外的 slice。

### 练习题：以下代码会输出什么？

```go
package main

import "fmt"

func main() {
	s := make([]int, 3)
	s = append(s, 1, 2, 3)
	fmt.Println(s)
}
```

**答**：`[0 0 0 1 2 3]`

## Map

### Go 中使用 map 的注意事项

1. **Map 不是并发安全的**。官方的 [faq](https://go.dev/doc/faq#atomic_maps) 里有说明，考虑到有性能损失，map 没有设计成原子操作，在并发读写时会有问题。
   1. 可以使用 `sync.RWMutex` 来保护 `map`。
   2. `sync.Map` 是官方出品的并发安全的 map。他在内部使用了大量的原子操作来存取键和值，并使用了 `read` 和 `dirty` 两个原生 map 作为存储介质，具体实现流程可阅读相关源码。
2. **Map 的元素不可取址**。当 map 的元素为结构体类型的值时，无法直接修改结构体中的字段值。如果要修改，有两种方法：
   1. 使用结构体的指针类型 `map[string]*type`。
   2. 取出结构体，修改后设置回去。

### Go 中多次 range 循环 map 的顺序一样么？

不一样。多次运行，顺序是随机的。在 [mapiterinit](https://github.com/golang/go/blob/8d68b388d4d1debec8d349adac58dd9f1cb03d25/src/runtime/map.go#L816) 方法中获取了随机数，保证了每次遍历顺序的不一样。[官方博客](https://go.dev/blog/maps#iteration-order)中也提到了这一点。

测试代码如下：

```Go
package main

import (
	"fmt"
)

func main() {
	m := map[string]string{
		"1": "v1",
		"2": "v2",
		"3": "v3",
	}

	fmt.Println("First:")
	for k, v := range m {
		fmt.Println(k, v)
	}
	fmt.Println("\nSecond:")
	for k, v := range m {
		fmt.Println(k, v)
	}
}
```

输出为：

```
First:
2 v2
3 v3
1 v1

Second:
3 v3
1 v1
2 v2
```
