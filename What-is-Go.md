# What is Go?

## 1. Hello, world!

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("你好，世界！")
}
```

1. Go 代码是使用包来组织的，包类似于库和模块。每个源文件都包含 package 声明，指明这个源文件属于哪个包。
1. import 指明被导入的其他包的列表，支持别名。
1. Go 原生地支持 Unicode，可以处理所有国家的语言。

运行程序的方法一般有二：

* 最简单的子命令：`run`，编译、链接源文件，运行生成的可执行文件。
* 运行 `build`，编译输出成可复用程序。

```bash
$ go run main.go
你好，世界！

$ go build main.go
$ ./main
你好，世界！
```

## 2. 复合数据类型之切片

### 2.1. 数组 vs 切片

* 数组是具有固定长度且拥有零个或者多个相同数据类型元素的序列。数组因为长度固定，所以很少直接使用。
* 切片（slice）表示一个拥有相同类型元素的可变长度的序列。切片的长度可以伸缩，比数组使用得更多。

切片是一种轻量级的数据结构，用来访问数组的元素。切片有3个属性：指向底层数组被引用首元素的指针、长度和容量。同一底层数组可以对应多个切片。

### 2.2. `append` 函数

`append` 是内置函数之一，用来将元素追加到切片的后面。在执行 `append` 函数时，如果底层数组容量足够，那么切片将被更新，引用同一底层数组；否则将创建新的底层数组来存储新元素，`append` 函数的增长策略是复杂的。

```go
var x []int
x = append(x, 1)            // 添加一个元素
x = append(x, 2, 3)         // 添加多个元素
x = append([]int{4}, x...)  // 添加可变长度参数列表
fmt.Println(x)              // [4 1 2 3]
```

### 2.3. 切片就地修改

`rotate` 和 `reverse` 函数可以就地修改切片元素。

### 2.4. 练习

1. https://leetcode-cn.com/problems/daily-temperatures/


## 3. 延迟函数调用、宕机和恢复

* `defer` 语句是一个普通的函数或方法调用。在正常情况下，`defer` 语句在 `return` 语句之后执行；在宕机等不正常情况下，实际的延迟函数调用将被推迟到包含 `defer` 语句的函数结束后才执行。这意味着 `defer` 语句可以更新函数的结果变量。
* Go 语言运行时（runtime）检测到错误将发生宕机。此时，正常的程序执行将终止，所有 `defer` 语句将执行，然后程序将异常退出。
* 处理宕机的方法有二：退出程序、进行恢复。后者通过内置的 `recover` 函数实现。当 `recover` 函数在 `defer` 语句的内部调用时，且包含 `defer` 语句的函数发生宕机，`recover` 函数将终止当前的宕机状态并返回宕机的值，函数将正常返回。在其他任何情况下，`recover` 函数无任何效果且返回 `nil`。

```go
package main

import (
	"fmt"
)

func main() {
	defer func() {
		switch r := recover(); r {
		case nil:
			fmt.Println("not panic")
			break
		// TODO 预期的宕机
		default:
			panic(r) // 未预期的宕机；继续宕机过程
		}
	}()
	// 其他 defer 语句，倒序调用
	fmt.Println("ok")
}
```

在正常情况下，延迟（匿名）函数调用发生在其他语句执行之后，`recover` 函数无任何效果，返回 `nil`，输出如下：

```
ok
not panic
```

## 4. 协程和通道

### 4.1. goroutine

在 Go 里，每一个并发执行的活动称为 goroutine，它类似于线程，但它是用户态的。

```go
package main

import (
	"fmt"
)

var (
	numberOfGoroutine = 4
)

func printInt(x int) {
	fmt.Println(x)
}

func main() {
	for i := 0; i < numberOfGoroutine; i++ {
		go printInt(i)
	}
	fmt.Println("ok")
}
```

一般地，在主 goroutine 退出时，其他 goroutine 未执行完毕，无打印值，输出如下：

```
ok
```

这个问题属于并发安全范畴，我们应该避免 goroutine 不受控制；`sync.WaitGroup` 类型是解决方案之一。

```go
package main

import (
	"fmt"
	"sync"
)

var (
	numberOfGoroutine = 4
)

func printInt(x int, wg *sync.WaitGroup) {
	fmt.Println(x)
	wg.Done()
}

func main() {
	var wg sync.WaitGroup
	wg.Add(numberOfGoroutine)
	for i := 0; i < numberOfGoroutine; i++ {
		go printInt(i, &wg)
	}
	wg.Wait()
	fmt.Println("ok")
}
```

这样，我们使循环体内部的 goroutine 在主 goroutine 退出时执行完毕，但执行顺序难以预测，详情参考 GMP 调度模型。一种合法的输出如下：

```
1
2
3
0
ok
```

### 4.2. 通道

通道（channel）是 goroutine 之间的连接，是可以让一个 goroutine 发送特定类型值到另一个 goroutine 的通信机制。

* 无缓冲通道上的发送操作将会阻塞，直到另一个 goroutine 在对应的通道上执行接收操作，这时值传送完成，两个 goroutine 都可以继续执行。相反，如果接收操作先执行，接收方 goroutine 将阻塞，直到另一个 goroutine 在同一个通道上发送一个值。无缓冲通道是同步的。
* 缓冲通道有一个元素队列，有最大长度。无缓冲通道的长度是0，而非1。
* 通道可以是单向的。

```go
package main

import (
	"fmt"
	"strconv"
)

var (
	numberOfGoroutine = 4
)

func getOpposite(x int, channelOfResult *chan int) {
	*channelOfResult <- -x
}

func main() {
	channelOfResult := make(chan int, numberOfGoroutine)
	for i := 0; i < numberOfGoroutine; i++ {
		go getOpposite(i, &channelOfResult)
	}
	for i := 0; i < numberOfGoroutine; i++ {
		fmt.Println(strconv.Itoa(<-channelOfResult) + " output")
	}
	fmt.Println("ok")
}
```

相比调用 `sync.WaitGroup` 类型控制 goroutine，调用通道，我们可以知道 goroutine 执行完毕的顺序。这个程序的输出是难以预测的，一种合法的输出如下：

```
-1 output
-2 output
-3 output
0 output
ok
```

## 5. 深入理解 Go，我建议你……

1. 抽象地理解 Go 的关键特性，例如 GMP 调度模型、垃圾回收。
1. 阅读 Go 的代码，适合新手的：切片、`error` 等基本数据类型是怎样定义的？
1. Go 的语法严格、简洁，理解常见的设计模式将对你阅读开源项目大有裨益。
1. Go 很适合后端开发，熟悉一些常见的框架将提升你阅读和编写代码的效率，例如 [`cobra`](https://github.com/spf13/cobra)、[`gin`](https://github.com/gin-gonic/gin)。