---
title: "泛型设计"
type: docs
---

# 泛型设计

大家好，我是煎鱼。

Go1.18 的泛型是闹得沸沸扬扬，如今已经基本成型，就由煎鱼带大家一起摸透 Go 泛型。

本文内容主要涉及泛型的 3 大设计理念，4 大元素，值得大家深入学习。

分别是：
- 设计理念：
	- 类型参数。
	- 类型约束。
	- 类型推导。
- 语法元素：
	- 联合元素。
	- 近似元素。
	- 嵌入约束。
	- 接口类型。

## 类型参数

类型参数，这个名词。不熟悉的小伙伴咋一看就懵逼了。

泛型代码是使用抽象的数据类型编写的，我们将其称之为类型参数。当程序运行通用代码时，类型参数就会被类型参数所取代。也就是**类型参数是泛型的抽象数据类型**。

简单的泛型例子：

```go

func Print(s []T) {
	for _, v := range s {
		fmt.Println(v)
	}
}
```

代码有一个 `Print` 函数，它打印出一个片断的每个元素，其中片断的元素类型，这里称为 T，是未知的。

这里引出了一个要做泛型语法设计的点，那就是：T 的**泛型类型参数，应该如何定义**？

在现有的设计中，分为两个部分：
- 类型参数列表：**类型参数列表将会出现在常规参数的前面**。为了区分类型参数列表和常规参数列表，类型参数列表**使用方括号**而不是小括号。
- 类型参数约束：如同常规参数有类型一样，类型参数也有元类型，被称为约束（后面会进一步介绍）。

结合完整的例子如下：

```go
// Print 可以打印任何片断的元素。
// Print 有一个类型参数 T，并有一个单一的（非类型）的 s，它是该类型参数的一个片断。
func Print[T any](s []T) {
	// do something...
}
```

在上述代码中，我们声明了一个函数 `Print`，其有一个类型参数 T，类型约束为 `any`，表示为任意的类型，作用与 `interface{}` 一样。他的入参变量 `s` 是类型 T 的切片。

函数声明完了，在函数调用时，我们需要指定类型参数的类型。如下：

```go
	Print[int]([]int{1, 2, 3})
```

在上述代码中，我们指定了传入的类型参数为 int，并传入了 `[]int{1, 2, 3}` 作为参数。

其他类型，例如 float64:

```go
	Print[float64]([]float64{0.1, 0.2, 0.3})
```

也是类似的声明方式，照着套就好了。

## 类型约束

说完类型参数，我们再说说 “约束”。在所有的类型参数中都要指定类型约束，才能叫做完整的泛型。

以下分为两个部分来具体展开讲解：
- 定义函数约束。
- 定义运算符约束

### 为什么要有类型约束

为了**确保调用方能够满足接受方的程序诉求**，保证程序中所应用的函数、运算符等特性能够正常运行。

泛型的类型参数，类型约束，相辅相成。

### 定义函数约束

#### 问题点

我们看看 Go 官方所提供的例子：

```go
func Stringify[T any](s []T) (ret []string) {
	for _, v := range s {
		ret = append(ret, v.String()) // INVALID
	}
	return ret
}
```

该方法的实现目的是：任何类型的切片都能转换成对应的字符串切片。但程序逻辑里有一个问题，那就是他的入参 T 是 `any` 类型，是任意类型都可以传入。

其内部又调用了 `String` 方法，自然也就会报错，因为只像是 int、float64 等类型，就可能没有实现该方法。

你说要定义有效的类型约束，那像是上面的例子，在泛型中如何实现呢？

要求传入方要有内置方法，就得定义一个 `interface` 来约束他。

#### 单个类型

例子如下：

```go
type Stringer interface {
	String() string
}
```

在泛型方法中应用：

```go
func Stringify[T Stringer](s []T) (ret []string) {
	for _, v := range s {
		ret = append(ret, v.String())
	}
	return ret
}
```

再将 `Stringer` 类型放到原有的 `any` 类型处，就可以实现程序所需的诉求了。

#### 多个类型

如果是多个类型约束。例子如下：

```go
type Stringer interface {
	String() string
}

type Plusser interface {
	Plus(string) string
}

func ConcatTo[S Stringer, P Plusser](s []S, p []P) []string {
	r := make([]string, len(s))
	for i, v := range s {
		r[i] = p[i].Plus(v.String())
	}
	return r
}
```

与常规的入参、出参类型声明一样的规则。

### 定义运算符约束

完成了函数约束的定义后，剩下一个要啃的大骨头就是 “运算符” 的约束了。

#### 问题点

我们看看 Go 官方的例子：

```go
func Smallest[T any](s []T) T {
	r := s[0] // panic if slice is empty
	for _, v := range s[1:] {
		if v < r { // INVALID
			r = v
		}
	}
	return r
}
```

经过上面的函数例子，我们很快能意识到这个程序根本无法运行成功。

其入参是 `any` 类型，程序内部是按 slice 类型来获取值，且在内部又进行运算符比较，那如果真是 slice，内部就可能每个值类型都不一样。

如果一个是 slice，一个是 int 类型，又如何进行运算符的值对比？

#### 近似元素

可能有的同学想到了重载运算符，但...想太多了，Go 语言没有支持的计划。为此做了一个新的设计，那就是允许限制类型参数的类型范围。

语法如下：

```go
InterfaceType  = "interface" "{" {(MethodSpec | InterfaceTypeName | ConstraintElem) ";" } "}" .
ConstraintElem = ConstraintTerm { "|" ConstraintTerm } .
ConstraintTerm = ["~"] Type .
```

例子如下：

```go
type AnyInt interface{ ~int }
```

上述声明的类型集是 `~int`，也就是所有类型为 int 的类型（如：int、int8、int16、int32、int64）都能够满足这个类型约束的条件。

包括底层类型是 int8 类型的，例如：

```go
type AnyInt8 int8
```

也就是在该匹配范围内的。

#### 联合元素

如果希望进一步缩小限定类型，可以结合分隔符来使用，用法为：

```go
type AnyInt interface{
 ~int8 | ~int64
}
```

就可以将类型集限定在 int8 和 int64 之中。

#### 实现运算符约束

基于新的语法，结合新的概念联合和近似元素，可以把程序改造一下，实现在泛型中的运算符的匹配。

类型约束的声明，如下：

```go
type Ordered interface {
	~int | ~int8 | ~int16 | ~int32 | ~int64 |
		~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr |
		~float32 | ~float64 |
		~string
}
```

应用的程序如下：

```go
func Smallest[T Ordered](s []T) T {
	r := s[0] // panics if slice is empty
	for _, v := range s[1:] {
		if v < r {
			r = v
		}
	}
	return r
}
```

确保了值均为基础数据类型后，程序就可以正常运行了。

## 类型推导

程序员写代码，一定程度的偷懒是必然的。

在一定的场景下，可以通过类型推导来避免明确地写出一些或所有的类型参数，编译器会进行自动识别。

建议复杂函数和参数能明确是最好的，否则读代码的同学会比较麻烦，可读性和可维护性的保证也是工作中重要的一点。

### 参数推导

函数例子。如下：

```go
func Map[F, T any](s []F, f func(F) T) []T { ... }
```

公共代码片段。如下：

```go
var s []int
f := func(i int) int64 { return int64(i) }
var r []int64
```
明确指定两个类型参数。如下：

```go
r = Map[int, int64](s, f)
```

只指定第一个类型参数，变量 f 被推断出来。如下：

```go
r = Map[int](s, f)
```

不指定任何类型参数，让两者都被推断出来。如下：

```go
r = Map(s, f)
```

### 约束推导

神奇的在于，类型推导不仅限与此，连约束都可以推导。

函数例子，如下：

```go
func Double[E constraints.Number](s []E) []E {
	r := make([]E, len(s))
	for i, v := range s {
		r[i] = v + v
	}
	return r
}
```

基于此的推导案例，如下：

```go
type MySlice []int

var V1 = Double(MySlice{1})
```

MySlice 是一个 int 的切片类型别名。变量 V1 的类型编译器推导后 []int 类型，并不是 MySlice。

原因在于编译器在比较两者的类型时，会将 MySlice 类型识别为 []int，也就是 int 类型。

要实现 “正确” 的推导，需要如下定义：

```go
type SC[E any] interface {
	[]E 
}

func DoubleDefined[S SC[E], E constraints.Number](s S) S {
	r := make(S, len(s))
	for i, v := range s {
		r[i] = v + v
	}
	return r
}
```

基于此的推导案例。如下：

```go
var V2 = DoubleDefined[MySlice, int](MySlice{1})
```

只要定义显式类型参数，就可以获得正确的类型，变量 V2 的类型会是 MySlice。

那如果不声明约束呢？如下：

```go
var V3 = DoubleDefined(MySlice{1})
```

编译器通过函数参数进行推导，也可以明确变量 V3 类型是 MySlice。

## 关键名词

原本 @Ian Lance Taylor 设计的的泛型类型关键字如下：

```golang
type T interface {
 type int, int8, int16, int32, int64, uint, uint8, uint16, uint32, uint64, uintptr, float32, float64, complex64, complex128, string
}
```
看起来好像非常 “顺眼”。但在《[proposal: Go 2: sum types using interface type lists](https://github.com/golang/go/issues/41716 "proposal: Go 2: sum types using interface type lists")》中社区进行了热烈的讨论。

认为该类型约束的关键字，过于 “模棱两可”。像是 @Damien Neil 所提出的以下两个例子。

结构体的例子：

```golang
package p
type mustIncludeDefaultCase struct{}
type MySum interface {
  type int, float64, mustIncludeDefaultCase
}
```

不明确的点之一，如果类型列表包含一个未导出的类型，那又应该是如何处理呢？

接口的例子：

```golang
type T interface { type int16, int32 }
func main() {
  var x T
  switch x.(type) {
  case int16:
  case int32:
  } 
}
```

你认为程序会跑进哪个 switch-case 的代码块里呢，是 int16，还是 int32？

不，都不会，变量 x 是 nil，如此迷惑。

在社区讨论中，发现设计与真实场景一结合，发现这个类型规则在普通的接口类型、在约束中使用也太微妙了。

用类型列表嵌入接口时的行为也很奇怪。认为可以做的更好，那就是 “更显式”。

为此，Go 泛型的设计者 @Ian Lance Taylor 提出了一个新的提案《[spec: generics: use type sets to remove type keyword in constraints](https://github.com/golang/go/issues/45346 "spec: generics: use type sets to remove type keyword in constraints")》。

其包含三个新的、更简单的想法来取代泛型提案中定义的类型列表，成为了正式泛型语法。

### 语法定义

新语法在泛型处增加一个新概念：接口元素（interface elements），用作约束条件的接口类型，或者被嵌入约束条件的接口类型，允许嵌入一些额外的构造。

被嵌入的可以是：
- 任何类型，而不仅仅是一个接口类型。
- 一个新的句法结构，称为近似元素。
- 一个新的句法结构，称为联合元素。

重点名词，我们继续展开讲解，分别是：
- 嵌入约束。
- 近似元素。
- 联合元素。
- 接口类型。

### 联合元素

原先的语法中，类型约束会用逗号分隔的方式来展示。

如下：
```golang
type int, int8, int16, int32, int64
``` 

在新语法中，结合定义为 union element（联合元素），写成一系列由竖线 ”|“ 分隔的类型或近似元素。

如下：

```golang
type PredeclaredSignedInteger interface {
	int | int8 | int16 | int32 | int64
}
```

常常会和下面讲到的近似元素一起使用。

### 近似元素

新语法，他的标识符是 “~”，完整用法是 `~T`。`~T` 是指底层类型为 T 的所有类型的集合。例如：

```golang
type AnyInt interface{ ~int }
```

他的类型集是 `~int`，也就是所有类型为 int 的类型（如：int、int8、int16、int32、int64）都能够满足这个类型约束的条件，包括底层类型为 int 类型的（例如：类型别名）。

再结合以上的分隔来使用，用法为：

```golang
type SignedInteger interface {
	~int | ~int8 | ~int16 | ~int32 | ~int64
}
```

相当于泛型提案中使用的以下类型：

```golang
interface {
	type int, int8, int16, int32, int64
}
```

新语法只需借助近似标识符 `~int` 来表达就可以了，更明确的表示了近似匹配，而不是存在隐式理解。

### 嵌入约束

一个类型约束可以嵌入另一个约束，联合元素可以包括约束。

例如：

```golang
// Signed is a constraint whose type set is any signed integer type.
type Signed interface {
	~int | ~int8 | ~int16 | ~int32 | ~int64
}

// Unsigned is a constraint whose type set is any unsigned integer type.
type Unsigned interface {
	~uint | ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uintptr
}

// Float is a constraint whose type set is any floating point type.
type Float interface {
	~float32 | ~float64
}

// Ordered is a constraint whose type set is any ordered type.
// That is, any type that supports the < operator.
type Ordered interface {
	Signed | Unsigned | Float | ~string
}
```

这个很好理解，就是正式支持嵌入约束了。

### 接口类型（联合约束元素）

在联合元素中，使用接口类型的话。将会把该类型集添加到联合中。

例如：

```golang
type Stringish interface {
	string | fmt.Stringer
}
```

Stringish 的类型集将是字符串类型和所有实现 `fmt.Stringer` 的类型，任何这些类型（包括 `fmt.Stringer` 本身）将被允许作为这个约束的类型参数。
 
也就是针对接口类型做了特殊的处理。


## 总结

今天我们在文章中给大家介绍了泛型的三个重要概念，分别是：
- 类型参数：泛型的抽象数据类型。
- 类型约束：确保调用方能够满足接受方的程序诉求。
- 类型推导：避免明确地写出一些或所有的类型参数。

在内容中也涉及到了联合元素、近似元素、函数约束、运算符约束等新概念。本质上都是基于三个大概念延伸出来的新解决方法，一环扣一环。

你学会 Go 泛型了吗，设计的如何，欢迎一起讨论：）

## 参考
- [Type Parameters Proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md)
- [Summary of Go Generics Discussions](https://docs.google.com/document/d/1vrAy9gMpMoS3uaVphB32uVXX4pi-HnNjkMEgyAHX4N4/)
