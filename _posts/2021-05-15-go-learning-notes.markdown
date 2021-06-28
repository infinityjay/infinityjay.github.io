---
layout: post
title: go语法笔记
---

- [常用函数](#常用函数)
- [数据结构](#数据结构)
  - [数组](#数组)
  - [切片-动态数组](#切片-动态数组)
  - [Map哈希表](#map哈希表)
  - [字符串](#字符串)
  - [list列表](#list列表)
  - [heap堆](#heap堆)
- [语法](#语法)
  - [Slice](#slice)
  - [变量](#变量)
  - [声明](#声明)
  - [类型](#类型)
  - [语句](#语句)
    - [if语句](#if语句)
    - [swith语句](#swith语句)
    - [for 语句](#for-语句)
    - [for-range 语句](#for-range-语句)
    - [go 语句](#go-语句)
    - [select 语句](#select-语句)
    - [break语句](#break语句)
    - [goto语句](#goto语句)
    - [fallthrough语句](#fallthrough语句)
    - [defer语句](#defer语句)
- [语言特性](#语言特性)
  - [函数](#函数)
  - [接口](#接口)
- [并发](#并发)

# 常用函数

```go
//创建切片使用make
func make([]T, len, cap) []T

//复制函数，将src复制到dst
func copy(dst, src []T) int

//在分片末尾增加元素
func append(s []T, x ...T) []T

```

# 数据结构

## 数组

Go 语言数组在初始化之后大小就无法改变，存储元素类型相同、但是大小不同的数组类型在 Go 语言看来也是完全不同的，只有两个条件都相同才是同一类型。

* 初始化

Go 语言的数组有两种不同的创建方式，一种是显式的指定数组大小，另一种是使用 `[...]T` 声明数组，Go 语言会在编译期间通过源代码推导数组的大小：

```go
arr1 := [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3}
```

上述两种声明方式在运行期间得到的结果是完全相同的，后一种声明方式在编译期间就会被转换成前一种，这也就是编译器对数组大小的推导。

* 访问和赋值

无论是在栈上还是静态存储区，数组在内存中都是一连串的内存空间，我们通过指向数组开头的指针、元素的数量以及元素类型占的空间大小表示数组。如果我们不知道数组中元素的数量，访问时可能发生越界；而如果不知道数组中元素类型的大小，就没有办法知道应该一次取出多少字节的数据，无论丢失了那个信息，我们都无法知道这片连续的内存空间到底存储了什么数据：

## 切片-动态数组

在 Go 语言中，切片类型的声明方式与数组有一些相似，不过由于切片的长度是动态的，所以声明时只需要指定切片中的元素类型：

```go
[]int
[]interface{}
```

从切片的定义我们能推测出，切片在编译期间的生成的类型只会包含切片中的元素类型，即 `int` 或者 `interface{}` 等。

* 初始化

Go 语言中包含三种初始化切片的方式：

1. 通过下标的方式获得数组或者切片的一部分；
2. 使用字面量初始化新的切片；
3. 使用关键字 `make` 创建切片：

```go
arr[0:3] or slice[0:3]
slice := []int{1, 2, 3}
slice := make([]int, 10)
```

使用下标

使用下标创建切片是最原始也最接近汇编语言的方式，它是所有方法中最为底层的一种，编译器会将 `arr[0:3]` 或者 `slice[0:3]` 等语句转换成 `OpSliceMake` 操作，我们可以通过下面的代码来验证一下：

```go
// ch03/op_slice_make.go
package opslicemake

func newSlice() []int {
	arr := [3]int{1, 2, 3}
	slice := arr[0:1]
	return slice
}
```



![2020-03-12-15839729948451-golang-slice-append](https://img.draveness.me/2020-03-12-15839729948451-golang-slice-append.png)





## Map哈希表

* 初始化

字面量 

目前的现代编程语言基本都支持使用字面量的方式初始化哈希，一般都会使用 `key: value` 的语法来表示键值对，Go 语言中也不例外：

```go
hash := map[string]int{
	"1": 2,
	"3": 4,
	"5": 6,
}
```

我们需要在初始化哈希时声明键值对的类型，这种使用字面量初始化的方式最终都会通过 [`cmd/compile/internal/gc.maplit`](https://draveness.me/golang/tree/cmd/compile/internal/gc.maplit) 初始化，我们来分析一下该函数初始化哈希的过程：

```go
func maplit(n *Node, m *Node, init *Nodes) {
	a := nod(OMAKE, nil, nil)
	a.Esc = n.Esc
	a.List.Set2(typenod(n.Type), nodintconst(int64(n.List.Len())))
	litas(m, a, init)

	entries := n.List.Slice()
	if len(entries) > 25 {
		...
		return
	}

	// Build list of var[c] = expr.
	// Use temporaries so that mapassign1 can have addressable key, elem.
	...
}
```

当哈希表中的元素数量少于或者等于 25 个时，编译器会将字面量初始化的结构体转换成以下的代码，将所有的键值对一次加入到哈希表中：

```go
hash := make(map[string]int, 3)
hash["1"] = 2
hash["3"] = 4
hash["5"] = 6
```

这种初始化的方式与的[数组](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array/)和[切片](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array-and-slice/)几乎完全相同，由此看来集合类型的初始化在 Go 语言中有着相同的处理逻辑。

一旦哈希表中元素的数量超过了 25 个，编译器会创建两个数组分别存储键和值，这些键值对会通过如下所示的 for 循环加入哈希：

```go
hash := make(map[string]int, 26)
vstatk := []string{"1", "2", "3", ... ， "26"}
vstatv := []int{1, 2, 3, ... , 26}
for i := 0; i < len(vstak); i++ {
    hash[vstatk[i]] = vstatv[i]
}
```

这里展开的两个切片 `vstatk` 和 `vstatv` 还会被编辑器继续展开，具体的展开方式可以阅读上一节了解[切片的初始化](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array-and-slice/)，不过无论使用哪种方法，使用字面量初始化的过程都会使用 Go 语言中的关键字 `make` 来创建新的哈希并通过最原始的 `[]` 语法向哈希追加元素。

运行时 

当创建的哈希被分配到栈上并且其容量小于 `BUCKETSIZE = 8` 时，Go 语言在编译阶段会使用如下方式快速初始化哈希，这也是编译器对小容量的哈希做的优化：

```go
var h *hmap
var hv hmap
var bv bmap
h := &hv
b := &bv
h.buckets = b
h.hash0 = fashtrand0()
```

* 读写操作

哈希表作为一种数据结构，我们肯定要分析它的常见操作，首先就是读写操作的原理。哈希表的访问一般都是通过下标或者遍历进行的：

```go
_ = hash[key]

for k, v := range hash {
    // k, v
}
```

这两种方式虽然都能读取哈希表的数据，但是使用的函数和底层原理完全不同。前者需要知道哈希的键并且一次只能获取单个键对应的值，而后者可以遍历哈希中的全部键值对，访问数据时也不需要预先知道哈希的键。在这里我们会介绍前一种访问方式，第二种访问方式会在 `range` 一节中详细分析。

数据结构的写一般指的都是增加、删除和修改，增加和修改字段都使用索引和赋值语句，而删除字典中的数据需要使用关键字 `delete`：

```go
hash[key] = value
hash[key] = newValue
delete(hash, key)
```

访问：

```go
v     := hash[key] // => v     := *mapaccess1(maptype, hash, &key)
v, ok := hash[key] // => v, ok := mapaccess2(maptype, hash, &key)
```

赋值语句左侧接受参数的个数会决定使用的运行时方法：

- 当接受一个参数时，会使用 [`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1)，该函数仅会返回一个指向目标值的指针；
- 当接受两个参数时，会使用 [`runtime.mapaccess2`](https://draveness.me/golang/tree/runtime.mapaccess2)，除了返回目标值之外，它还会返回一个用于表示当前键对应的值是否存在的 `bool` 值：





## 字符串



## list列表

* 初始化

1. 通过 container/list 包的`New() 函数`初始化 list

> 变量名 := list.New()

1. 通过 `var 关键字`声明初始化 list

> var 变量名 list.List

* 常用操作

```go
package main
import "container/list"
func main() {
    l := list.New()
    // 尾部添加
    l.PushBack("canon")
    // 头部添加
    l.PushFront(67)
    // 尾部添加后保存元素句柄
    element := l.PushBack("fist")
    // 在fist之后添加high
    l.InsertAfter("high", element)
    // 在fist之前添加noon
    l.InsertBefore("noon", element)
    // 使用
    l.Remove(element)
}
```

![img](https://upload-images.jianshu.io/upload_images/13868689-ed3351ffc0af1c07.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/846/format/webp)

第一个元素：l.Front()

最后一个元素：l.Back()

结点e的值：e.Value  e.Value.(int)

合并两个链表

```go
l2.PushBackList(l2)
```

* 遍历列表

遍历双链表需要配合 Front() 函数获取头元素，遍历时只要元素不为空就可以继续进行，每一次遍历都会调用元素的 Next() 函数，代码如下所示。

```go
l := list.New()
// 尾部添加
l.PushBack("canon")
// 头部添加
l.PushFront(67)
for i := l.Front(); i != nil; i = i.Next() {
    fmt.Println(i.Value)
}
```

代码输出如下：

```undefined
67
canon
```

## heap堆

* 导入包

```go
package main
import (
  "container/heap"
  "fmt"
)
```

堆使用的数据结构是最小二叉树，根节点比左子树和右子树的所有值都小

* heap提供的方法

```go
h := &IntHeap{3, 8, 6}  // 创建IntHeap类型的原始数据
func Init(h Interface)  // 对heap进行初始化，生成小根堆（或大根堆）
func Push(h Interface, x interface{})  // 往堆里面插入内容
func Pop(h Interface) interface{}  // 从堆顶pop出内容
func Remove(h Interface, i int) interface{}  // 从指定位置删除数据，并返回删除的数据
func Fix(h Interface, i int)  // 从i位置数据发生改变后，对堆再平衡，优先级队列使用到了该方法
```





# 语法



## Slice

```go
//构建数组，大小为4
var a [4]int
a[0] = 1
i := a[0]
// i == 1

//make函数
func make([]T, len, cap) []T
//其中T代表要创建的分片的元素类型。make函数需要类型、长度和可选的容量。调用时，make会分配一个数组并返回一个指向该数组的分片:
var s []byte
s = make([]byte, 5, 5)
// s == []byte{0, 0, 0, 0, 0}

//分片
//分片也可以通过 “切片” 现有的分片或数组来形成。切片是通过指定一个半开放的范围，用冒号隔开两个指数来完成的。例如，表达式 b[1:4] 创建了一个包含b元素1到3的分片（生成的分片的指数为0到2）
b := []byte{'g', 'o', 'l', 'a', 'n', 'g'}
// b[1:4] == []byte{'o', 'l', 'a'}, sharing the same storage as b

//修改重新分片的元素会修改原始分片的元素
d := []byte{'r', 'o', 'a', 'd'}
e := d[2:]
// e == []byte{'a', 'd'}
e[1] = 'm'
// e == []byte{'a', 'm'}
// d == []byte{'r', 'o', 'a', 'm'}

//增长切片
//利用copy函数使s的容量增加一倍的操作
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t

//利用append函数在分片s末尾增加元素
a := make([]int, 1)
// a == []int{0}
a = append(a, 1, 2, 3)
// a == []int{0, 1, 2, 3}


```

## 变量

* 变量定义

  `var` 语句定义一个变量的列表；跟函数的参数列表一样，类型在后面。

  `var` 语句可以定义在包或函数级别。

* 短声明变量

  在函数中，`:=` 简洁赋值语句在明确类型的地方，可以用于替代 `var` 定义。

  ```go
  func main() {
  	var i, j int = 1, 2
  	k := 3
  	c, python, java := true, false, "no!"
  }
  ```

  函数外的每个语句都必须以关键字开始（`var`、`func`、等等），`:=` 结构不能使用在函数外。

* go语言规范

  ```go
  var x interface{}  // x 是 nil，它有一个静态类型 interface{}
  var v *T           // v 的值为 nil，静态类型为 *T
  x = 42             // x 的值为 42，动态类型为 int
  x = v              // x 的值为 (*T)(nil)，动态类型为 *T
  ```

## 声明

* iota

  在常量声明中，预先声明的标识符 iota 代表连续的非类型整数常量。它的值是该常量声明中各自ConstSpec的索引，从0开始。它可以用来构造一组相关的常量。

## 类型

* 布尔类型：bool

* 数字类型：

```
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32

//这一组预先声明的数值类型，其大小与实现有关
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value

```

* 字符串类型：string

* 数组类型：定义`var a [10]int` ,数组不能改变大小

```go
a := [2]string{"a", "b"}
a := []string{"a", "b"}
a := [...]string{"a", "b"}
```

* 切片类型：slice

`[]T` 是一个元素类型为 `T` 的 slice。

```go
p := []int{2, 3, 5, 7, 11, 13}
```

表达式`s[lo:hi]`表示从 `lo` 到 `hi-1` 的 slice 元素，含两端。因此`s[lo:lo]`是空的，而`s[lo:lo+1]`有一个元素。

注意：slice的长度可以在构造时通过参数明确指定，也可以在切片时通过上下两个下标计算而来。而容量则需要考虑左下标开始位置。

```go
func main() {
	a := make([]int, 5)	// 指定长度为5，容量没有设置，则和长度相同：len=5 cap=5
	printSlice("a", a)
	b := make([]int, 0, 5) // 指定长度为0，容量为5：len=0 cap=5
	printSlice("b", b)
	c := b[:2] // 切片时长度为下表差，容量计算时需要考虑左下标开始位置，这里的左下标从0开始：len=2 cap=5
	printSlice("c", c)
	d := c[2:5] // 切片时长度为下表差，容量计算时需要考虑左下标开始位置，这里的左下标从2开始：len=3 cap=5
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}
```

slice 的零值是 `nil`。一个 nil 的 slice 的长度和容量是 0。

Go 提供了一个内建函数 `append` 向 slice 添加元素：

```go
func append(s []T, vs ...T) []T
```

* struct类型

结构体是字段的集合。结构体定义的语法：

```go
type Vertex struct {
	X int
	Y int
}
```

* 指针类型

* 函数类型

* 接口类型

  接口类型指定了一个称为 interface/接口的方法集。接口类型的变量可以存储任何类型的值，其方法集是接口的任何超集。这样的类型被称为**实现**了接口。未初始化的接口类型变量的值是nil。

  接口类型可以通过方法规范明确地指定方法，也可以通过接口类型名称嵌入其他接口的方法。

  ```go
  // A simple File interface.
  interface {
  	Read([]byte) (int, error)
  	Write([]byte) (int, error)
  	Close() error
  }
  ```

* map类型

map 在使用之前必须用 `make` 而不是 `new` 来创建；值为 `nil` 的 map 是空的，并且不能赋值。

```go
m := make(map[string]string) // 语法是 "map[key的类型]value的类型"
m["key1"] = "value1"
fmt.Println(m["key1"])
```

或者直接通过指定key、value来创建：

```go
var m = map[string]string{
	"key1": "value1",
	"key2": "value2",
	"key3": "value3",	// 注意最后一行的结尾也必须有逗号
}
```

value的类型可以忽略，比如下面这种写法：

```go
var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
```

可以简化为：

```go
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

修改 map：

```go
func main() {
	m := make(map[string]int)

	m["Answer"] = 42	//在 map 中插入或修改
	fmt.Println("The value:", m["Answer"])  // 通过key获取值

	m["Answer"] = 48 //在 map 中插入或修改
	fmt.Println("The value:", m["Answer"])

	delete(m, "Answer") // 从 map 中删除key
	fmt.Println("The value:", m["Answer"])

	v, ok := m["Answer"] // 双赋值检测某个键存在，如果存在则第二个参数返回true
	fmt.Println("The value:", v, "Present?", ok)
}
```

初始容量并不约束其大小：为了容纳其中存储的项目数量map可以增长，但 nil map除外。nil map相当于一个空map，但不能添加任何元素。

* channel类型

通道为并发执行的函数提供了一种机制，通过发送和接收指定元素类型的值进行通信。未初始化的通道的值为nil。

```go
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

可选的 `<-` 操作符指定通道方向，发送或接收。如果没有给出方向，则通道是双向的。通道可以通过赋值或显式转换来限制只能发送或只能接收。

```go
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints
```

## 语句

* Send语句

https://golang.org/ref/spec#Send_statements

发送语句在通道上发送一个值。通道表达式必须是通道类型，通道方向必须允许发送操作，要发送的值的类型必须可以分配给通道的元素类型。

```
SendStmt = Channel "<-" Expression .
Channel  = Expression .
```

在通信开始之前，通道和值表达式都会被评估。通信会被阻塞，直到发送可以继续进行。如果接收者准备好了，在无缓冲通道上的发送就可以进行。在缓冲通道上的发送，如果缓冲区有空间，就可以进行。在已关闭通道上的发送会引起运行时恐慌。在nil通道上的发送会永远阻塞。

```go
ch <- 3  // send value 3 to channel ch
```

* 控制流程语句

### if语句

```go
if v := math.Pow(x, n); v < lim {
    // v在这里可以访问
    return v
}
// v在这里不可以访问
```

### swith语句

switch的语法和for、if类似，同样的括号和花括号使用规则，同样的容许在switch前执行一个简单的语句，同样的变量访问范围限制：

```go
switch os := runtime.GOOS; os {
    case "darwin":
    	fmt.Println("OS X.")
    case "linux":
    	fmt.Println("Linux.")
    default:
    	fmt.Printf("%s.", os)
}
```

golang中的switch在命中某个case子语句并执行完成之后，会自动终结分支并结束switch语句。这是默认行为，和c，java中会自动继续下一个分支匹配，需要明确break才能退出不同。

如果想继续执行后面的case子语句，需要在case子语句最后使用 `fallthrough` 语句。

### for 语句

Go 只有一种循环结构 `for` 循环。

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

也可以类似C、Java中的while循环：

```go
sum := 1
for sum < 1000 {
    sum += sum
}
```

### for-range 语句

带有 “range”子句的 “for” 语句会遍历数组、切片、字符串或map的所有条目，或通道（channel）上接收的值。对于每个条目，如果存在，它将迭代值分配给相应的迭代变量，然后执行该块。

如果你正在循环数组、切片、字符串或map，或者从一个通道读取，range子句可以管理循环。

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

如果你只需要range内的第一项（键或索引），就放弃第二项。

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

如果你只需要范围中的第二项（值），使用空白的标识符（下划线）来丢弃第一项。

```go
sum := 0
for _, value := range array {
    sum += value
}
```

### go 语句

“go”语句在同一地址空间内，以独立的并发控制线程或goroutine的形式开始执行一个函数调用。

### select 语句

select是Golang中的控制语句，语法类似于switch语句。但是select只用于通信，要求每个case必须是IO操作。

不带default语句的select会阻塞直到某个case满足：

```go
ch1 := make (chan int, 1)
ch2 := make (chan int, 1)

select {
case <-ch1:
    fmt.Println("ch1 pop one element")
case <-ch2:
    fmt.Println("ch2 pop one element")
}
```

如果两个case同时满足，则随机执行某个case语句，其他case语句不会执行。

如果不想阻塞，则可以带上default子语句：

```go
select {
case <-ch1:
    fmt.Println("ch1 pop one element")
case <-ch2:
    fmt.Println("ch2 pop one element")
default:
    fmt.Println("not ready yet")
}
```

如果两个case条件都不满足，则直接跳到 default 流程而不阻塞。

### break语句

“break “语句终止了同一函数中最内层的 “for”、”switch “或 “select “语句的执行。

```
BreakStmt = "break" [ Label ] .
```

如果有标签，则标签必须包围住 “for”、”switch “或 “select “语句，而且标签是执行终止。

### goto语句

“goto” 语句将控制权转移到同一函数中带有相应标签的语句。

```
GotoStmt = "goto" Label .
goto Error
```

执行 “goto”语句不能导致在goto之后有任何变量进入它还没有进行的范围。例如，这个例子：

```go
	goto L  // BAD
	v := 3
L:
```

是错误的，因为跳转到标签L时跳过了v的创建。

块外的 “goto”语句不能跳转到该块内的标签。例如，这个例子：

```go
if n%2 == 1 {
	goto L1
}
for n > 0 {
	f()
	n--
L1:
	f()
	n--
}
```

是错误的，因为标签L1在 “for”语句的块内，而goto却不在。

### fallthrough语句

“fallthrough”语句将控制权转移到 “switch” 语句中下一个case子句的第一条语句。它只能作为这种子句中的最后一个非空语句使用。

### defer语句

defer 语句会延迟函数的执行直到外层函数返回，通常用于执行清理操作。

```go
func CopyFile(dstName, srcName string) (written int64, err error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()

    dst, err := os.Create(dstName)
    if err != nil {
        return
    }
    defer dst.Close()

    return io.Copy(dst, src)
}
```

其他使用场景，如释放mutex：

```go
mu.Lock()
defer mu.Unlock()
```



# 语言特性

## 函数

* 多值返回

Go的一个不同寻常的特点是，**函数和方法可以返回多个值**。这种形式可以用来改进C程序中的几个笨拙的习语：带内错误返回，如-1代表EOF和修改按地址传递的参数。

* init 函数

除了不能用声明来表达的初始化之外，init函数的一个常见用途是在真正执行开始之前验证或修复程序状态的正确性。

```go
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

* 函数调用

参数传递：

传值：函数调用时会对参数进行拷贝，被调用方和调用方两者持有不相关的两份数据；

传引用：函数调用时会传递参数的指针，被调用方和调用方两者持有相同的数据，任意一方做出的修改都会影响另一方。

Go 语言选择了传值的方式，**无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝**。

如果希望函数内的变量能修改函数外的变量，可以传入变量的地址&，函数内以指针的方式操作变量。从效果上看类似引用。

* 方法

函数/Function：不属于任何结构体、类型，没有接收者

`go func Add(a, b int) int { return a + b }`

方法/Method：特定的结构体、类型关联，带有接收者

```go
type person struct {
    name string
}
        
func (p person) String() string{
    return "the person name is "+p.name
}
```

**函数（function）**接受一些参数作为输入，并产生一些输出。对于相同的输入，函数总是会产生相同的输出。这意味着它不依赖于状态。类型是作为函数的参数传递的。

Go中的**方法（Method）**是带有特定 receiver(type) 参数上的函数。它定义了类型的行为，它应该使用类型的状态。

Go语言里有两种类型的接收者：值接收者和指针接收者。

-- 使用值接收者: 在调用的时候，方法使用的其实是值接收者的一个副本，所以对该值的任何操作，不会影响原来的值接收者（或者说类型变量）。

-- 使用指针接收者：因为指针接收者传递的是一个指向原值指针的副本（即指针的副本），其指向的还是原来类型的值，所以修改时，同时也会影响原来值接收者（类型变量）的值。

总结：

在调用方法的时候，传递的接收者本质上都是副本，只不过可以是值的副本，也可以是指向这个值的指针的副本。指针具有指向原有值的特性，所以修改了指针指向的值，也就修改了原有的值。

* 闭包

闭包 = 函数 + 应用环境

对象是附有行为的数据，闭包是附有数据的行为。

## 接口

接口类型是由一组方法定义的集合。

```go
// 定义一个interface和它的方法
type Abser interface {
	Abs() float64
}

type Vertex struct {
	X, Y float64
}

// 让结构体实现interface要求的方法
func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

接口类型的值可以存放实现这些方法的任何值。

```go
var a Abser
v := Vertex{3, 4}
a = &v // *Vertex 实现了 Abser
```

# 并发

* 不要通过共享内存进行通信

在主流的编程语言中，当你想到代码的并发执行时，你大多会想到一堆线程并行运行，执行某种复杂的操作。那么，大多数情况下，你需要在不同的线程之间共享数据结构/变量/内存/什么的。你可以通过锁定这块内存来实现，这样就不会有两个线程同时访问/写入它，或者你只是让它自由漫游，并希望得到最好的结果。在很多流行的编程语言中，这通常是不同线程 “通信”的方式，这通常会导致各种问题，如竞争条件，内存管理 ，随机-奇怪-无法解释的异常-唤醒-你-整夜……等等。

* 而是通过通信来共享内存

那么Go是怎么做的呢？Go不锁定变量来共享内存，而是允许将存储在变量中的值从一个线程通信（或发送）到另一个线程（实际上，这并不完全是一个线程，但我们现在就把它当作一个线程）。默认的行为是发送数据的线程和接收数据的线程都会等待，直到值到达目的地。线程的 “等待”迫使线程之间在交换数据时进行适当的同步。在挽起袖子开始设计代码之前，先这样思考并发性问题；可以让软件更加稳定。

更明确的说：稳定性来自于这样一个事实 —— 默认情况下，发送线程和接收线程都不会做任何事情，直到值传输完成。意思是说，在另一个线程完成数据传输之前，任何一个线程对数据进行操作，都不会出现竞争条件或类似的问题。

Go提供了原生特性，你可以使用这些特性来实现这种行为，而不需要调用额外的类库或框架，这种行为只是简单地内置在语言中。如果你需要的话，Go还允许你拥有一个 “缓冲通道”。这意味着在某些情况下，你不希望两个线程都锁定或同步，直到一个值被传输，相反，你希望只有当你在两个线程之间的通道中填满了预定义数量的待处理值时，同步/锁定才会发生。

不过需要提醒的是，这种模式可能会被过度使用。你必须感觉到什么时候使用它，或者什么时候恢复到老式的共享内存模型。例如，引用计数最好在锁里面保护，文件访问也是如此。Go也会通过同步包在那里支持你。

* golang中的并发编码

在Go中，”goroutine”服务于我们上面描述的线程的概念。实际上，它并不是真正的线程，它基本上是一个函数，它可以和其他goroutine在同一个地址空间中并发运行。它们在O.S.线程之间是多路复用的，所以如果一个线程阻塞，其他线程可以继续运行。所有的同步和内存管理都是由Go原生完成的。它们之所以不是真正的线程，是因为它们不一定是一直并行的。然而，由于多路复用和同步，你会得到并发行为。要启动一个新的goroutine，你只需要使用关键字 “go”。

```go
go processdataFunction()
```

“go channel”是Go中实现并发的另一个关键概念。这是用于在goroutine之间进行内存通信的通道。要创建一个通道，可以使用 “make”。

```go
myChannel := make(chan int64)
```

创建一个缓冲通道，以便在goroutines等待之前允许更多的值被排队，看起来像这样：

```go
myBufferedChannel := make(chan int64,4)
```

在上面的两个例子中，我假设在这之前没有创建通道变量。这就是为什么我使用”:=“来创建具有推断类型的变量，而不是”=“，因为”=“只会赋值，而且如果之前没有声明该变量，将导致编译错误。

现在要使用一个通道，你可以使用”<-“符号。goroutine发送的值会像这样分配给通道。

```go
mychannel <- 54
```

接收数值的goroutine将从通道中提取数值，并将其分配到一个新的变量中，就像这样：

```go
myVar := <- mychannel
```

























