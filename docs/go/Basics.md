---
title: Basics
---

## For

[5 basic for loop patterns](https://yourbasic.org/golang/for-loop/)

- [Three-component loop](https://yourbasic.org/golang/for-loop/#three-component-loop)
- [While loop](https://yourbasic.org/golang/for-loop/#while-loop)
- [Infinite loop](https://yourbasic.org/golang/for-loop/#infinite-loop)
- [For-each range loop](https://yourbasic.org/golang/for-loop/#for-each-range-loop)
- [Exit a loop](https://yourbasic.org/golang/for-loop/#exit-a-loop)

## Range

[Go by example](https://gobyexample.com/range)

`range` on strings iterates over Unicode code points. The first value is the starting byte index of the `rune` and the second the `rune` itself. See [Strings and Runes](https://gobyexample.com/strings-and-runes) for more details.

[4 basic range loop (for-each) patterns](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/)

- [Basic for-each loop (slice or array)](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/#basic-for-each-loop-slice-or-array)
- [String iteration: runes or bytes](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/#string-iteration-runes-or-bytes)
- [Map iteration: keys and values](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/#map-iteration-keys-and-values)
- [Channel iteration](https://yourbasic.org/golang/for-loop-range-array-slice-map-channel/#channel-iteration)



## Defer Panic Revocer

[运行时异常和 panic](https://learnku.com/docs/the-way-to-go/132-runtime-exception-and-panic/3675)

`panic` 可以直接从代码初始化：当错误条件（我们所测试的代码）很严苛且不可恢复，程序不能继续运行时，可以使用 panic 函数产生一个中止程序的运行时错误。panic 接收一个做任意类型的参数，通常是字符串，在程序死亡时被打印出来。

[从 panic 中恢复（Recover）](https://learnku.com/docs/the-way-to-go/133-recovery-from-panic-recover/3676)

`recover` 只能在 defer 修饰的函数（参见 [6.4 节](https://go.learnku.com/docs/the-way-to-go/defer-and-tracking/45)）中使用：用于取得 panic 调用中传递过来的错误值，如果是**正常执行，调用 `recover` 会返回 nil**，且没有其它效果。

总结：panic 会导致栈被展开直到 defer 修饰的 recover () 被调用或者程序中止。

## Array and Slice

[Go Array](https://www.w3schools.com/go/go_arrays.php)

[Go Slice](https://www.w3schools.com/go/go_slices.php)

[区别](https://segmentfault.com/a/1190000013148775)

- `golang`中的数组是**`值类型`**,也就是说，如果你将一个数组赋值给另外一个数组，那么，实际上就是整个数组拷贝了一份
- 如果`golang`中的数组作为函数的参数，那么实际传递的参数是一份数组的拷贝，而不是数组的指针
- `array`的长度也是`Type`的一部分，这样就说明`[10]int`和`[20]int`是不一样的。
- `slice`是一个引用类型，是一个动态的指向数组切片的指针。
- `slice`是一个不定长的，总是指向底层的数组`array`的数据结构。

[slice append 后 capacity 增长的算法](https://studygolang.com/articles/18194)

1. 先将旧的slice容量乘以2，如果乘以2后的容量仍小于新的slice容量，则取新的slice容量(append多个elems)
2. 如果新slice小于等于旧slice容量的2倍，则取旧slice容量乘以2
3. 如果旧的slice容量大于1024，则新slice容量取旧slice容量乘以1.25

代码后面还会对newcap进行roundup，比如在64位平台，newcap是奇数的话就会+1

## Modules

[Go Modules Reference](https://go.dev/ref/mod)

Modules are how Go manages dependencies.

```bash
# The go mod init command initializes and writes a new go.mod file in the current directory, in effect creating a new module rooted at the current directory. The go.mod file must not already exist.
go mod init
# check the dependency
go mod why -m <module>
# adds missing module requirements necessary to build the current module’s packages and dependencies, and removes requirements on modules that don’t provide any relevant packages. It also adds any missing entries to go.sum and removes unnecessary entries.
go mod tidy
# The go mod vendor command constructs a directory named vendor in the main module’s root directory that contains copies of all packages needed to support builds and tests of packages in the main module
go mod vendor
```

[Create a Go module](https://go.dev/doc/tutorial/create-module)

test code for go module following the tut: [go-module](https://github.com/HesterG/go-study-notes/tree/main/codes/go-module)

[深入理解 Go Modules 的 go.mod 与 go.sum](https://cloud.tencent.com/developer/article/2020911)

[How to modify existing projects to use Go modules](https://jfrog.com/blog/converting-projects-for-go-modules/)

## Concurrency

[Concurrency in Go](https://www.youtube.com/watch?v=LvgVSSpwND8)

如果main执行完了，go routine有可能来不及做任何事就退出了。使用sleep或者`fmt.Scanln()`可以block main的执行。

开发中可以使用`sync.WaitGroup` 、`Channel`、`Select`等

### WaitGroup

![](https://user-images.githubusercontent.com/17645053/221389186-b0317baf-44d0-4303-914e-5ead3ae29083.png)

### Channel

![](https://user-images.githubusercontent.com/17645053/221389412-bfcb605a-8143-4a95-891d-9efb89a528b5.png)

### Select

![](https://user-images.githubusercontent.com/17645053/221389562-b68280ac-c184-4948-8e30-e6a0d04b3e11.png)

![](https://user-images.githubusercontent.com/17645053/221389776-2272f89f-7988-4030-88da-b22510e7c988.png)

[Concurrency and goroutines in golang](https://www.youtube.com/watch?v=V-0ifUKCkBI)

Thead vs. Goroutines

![](https://user-images.githubusercontent.com/17645053/221390127-1ee6fef8-8fb1-4ebd-b97a-d2c0bd4d2c1c.png)

[Go 并发编程｜并发、并行、Goroutine](https://juejin.cn/post/7094647362542534664)

## Context

[code example](https://github.com/HesterG/go-study-notes/blob/main/context.go)

