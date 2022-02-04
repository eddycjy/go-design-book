---
title: "泛型配套"
type: docs
---

# 泛型配套

大家好，我是煎鱼。

## maps

Go 泛型的配套标准库 [golang.org/x/exp/maps](https://cs.opensource.google/go/x/exp/+/master:maps/maps.go;bpv=0) 包已经正式提交，放出来了，可以使用。

![](https://image.eddycjy.com/5da1a2b500a65e1477beed26bdb16ed7.png)

如下图：

![cs.opensource.google](https://image.eddycjy.com/4bf04e171a997c508780744ffc6c66bf.png)

包代码如下：

```golang
package maps

func Keys[M ~map[K]V, K comparable, V any](m M) []K {
	r := make([]K, 0, len(m))
	for k := range m {
		r = append(r, k)
	}
	return r
}

func Values[M ~map[K]V, K comparable, V any](m M) []V {
	r := make([]V, 0, len(m))
	for _, v := range m {
		r = append(r, v)
	}
	return r
}

func Equal[M1, M2 ~map[K]V, K, V comparable](m1 M1, m2 M2) bool {
	if len(m1) != len(m2) {
		return false
	}
	for k, v1 := range m1 {
		if v2, ok := m2[k]; !ok || v1 != v2 {
			return false
		}
	}
	return true
}

func EqualFunc[M1 ~map[K]V1, M2 ~map[K]V2, K comparable, V1, V2 any](m1 M1, m2 M2, eq func(V1, V2) bool) bool {
	if len(m1) != len(m2) {
		return false
	}
	for k, v1 := range m1 {
		if v2, ok := m2[k]; !ok || !eq(v1, v2) {
			return false
		}
	}
	return true
}

func DeleteFunc[M ~map[K]V, K comparable, V any](m M, del func(K, V) bool) {
	for k, v := range m {
		if del(k, v) {
			delete(m, k)
		}
	}
}
```
- Keys：返回 map 的键值内容，键值将以不确定的顺序出现。
- Values：返回 map 的值，值将以不确定的顺序出现。
- Equal：检查两个地图是否包含相同的键/值对，内部会使用 `==` 来比较数值。
- EqualFunc：`EqualFunc`与 `Equal` 方法类似，但使用闭包方法来比较数值，键值仍然用 `==` 来比较。
- DeleteFunc：删除 map 中闭包方法返回 `true` 的任何键/值对。

```golang

func Clear[M ~map[K]V, K comparable, V any](m M) {
	for k := range m {
		delete(m, k)
	}
}

func Clone[M ~map[K]V, K comparable, V any](m M) M {
	r := make(M, len(m))
	for k, v := range m {
		r[k] = v
	}
	return r
}

func Copy[M ~map[K]V, K comparable, V any](dst, src M) {
	for k, v := range src {
		dst[k] = v
	}
}
```
- Clear：清除从 map 中删除所有条目，使之为空。
- Clone：返回一个 map 的副本，这是一个浅层克隆，新拷贝出来的的键和值使用普通的赋值来设置。
- Copy：复制 src 中的所有键/值对，并将其加入 dst。当 src 中的一个键已经存在于 dst 中时，dst 中的值将被与 src 中的键相关的值所覆盖。

## slices

## switch-type

### 背景

允许在 switch 语句中使用泛型时，能够进一步便捷的约束其类型参数。

例如：

```go
switch type T {
case A1:
case A2, A3:
   ...
}
```

也就是 switch-type 语句的 **T 类型可以是一个泛型的类型参**，case 所对应的的类型可以是任何类型，包括泛型的约束类型。

假设类型 T 的类型有可能是以下：

```go
interface{
    C
    A
}
```

可以借助泛型的近似元素来约束：

```go
    interface{
        C
        A1 | A2 | ... | An
    }
```

甚至还可以在 case 上有新的写法：

```go
case interface {~T}:
```

在支持泛型后，**switch 在 type 和 case 上会存在很多种可能性**，需要进行具体的特性支持，这个提案就是为此出现。

### 争议点

看到这里可能大家也想到了，这个味道很似曾相识，好像某个语法能够支持。因此，这个提案下最有争议的，就是与原有的类型断言的重复。

原有的类型断言如下：

```go
switch T.(type) {
case string:
   ...
default:
   ...
}
```

新的类型判别如下：

```go
switch type T {
case A1:
case A2, A3:
   ...
}
```

这么咋一看，其实类型断言的完全可以取代新的，那岂不是重复建设，造轮子了？

其实是没有完全取代的。差异点如下：

```go
type ApproxString interface { ~string }

func F[T ApproxString](v T "T ApproxString") {
    switch (interface{})(v).(type) {
    case string:
        fmt.Println(v)
    default:
        panic("脑子没进煎鱼")
    }
}

type MyString string

func main() {
    F(MyString("脑子进煎鱼了"))
}
```

看出来差别在哪了吗，答案是什么？

答案是：会抛出恐慌（panic）。

你可能纠结了，问题出在哪里？这传入的 ”脑子进煎鱼了“ 的类型是 `MyString`，他的基础类型是 `string` 类型，也满足 `ApproxString` 类型的近似类型 `~string` 的要求，怎么就不行了...

根本原因是因为他的类型是 interface，而非 string 类型。所以走到了 defalut 分支的恐慌。

### 案例

#### 多类型元素

```go
type Stringish interface {
	string | fmt.Stringer
}

func Concat[S Stringish](x []S "S Stringish") string {
    switch type S {
    case string:
        ...
    case fmt.Stringer:
        ...
    }
 }
```

类型 S 能够支持 string 和 fmt.Stringer 类型，case 配套对应实现。

#### 近似元素

```go
type Constraint interface {
    ~int | ~int8 | ~string
}

func ThisSyntax[T Constraint]( "T Constraint") {
    switch type T {
    case ~int | ~int8:
        ...
    case ~string:
        ...
    }
}

func IsClearerThanThisSyntax[T Constraint]( "T Constraint") {
    switch type T {
    case interface{~int | ~int8 }:
        ...
    case interface{ ~string }:
        ...
    }
}
```

类型 T 可能有很多类型，程序中用到了近似元素，也就是基础类型是 int、int8、string，这些类型中的任何一种都能够满足这个约束。

为此，switch-type 支持了，case 也要配套支持该特性。

### 总结

我相信原有的 `switch.(type)` 和 `switch type` 很大概率在 Go 底层会变成同一个逻辑块处理，再逐渐过渡。

这个提案的目的还是**为了解决若干引入泛型后，所带入的 BUG/需求**，正正是需要新的语法结构来解决的。

## 编译减慢

### 背景

在 Go1.18 已经正式释出正式的第一版。在网上 @danscales 进行了测试，提出的《cmd/compile: Go 1.18 compile time may be about 18% slower than Go.17 (largely from changes due to generics)》的问题。

表示在 Go1.18 起有了泛型后，编译速度将会变慢，虽然不意外，说明副作用还是有的，升级需谨慎。

以下为修整后概括的原文信息。

### 性能分析

这个测试主要是测试 Go 泛型对 Go 编译器带来的影响，并没有输入大量的测试用例，是最简单的比较，仅代表大部分的差异。

比较的内容是 Go 泛型的 -G=0 和 -G=3 模式下的编译时间。

分别代表以下含义：
- -G=0 模式：默认不打开泛型的模式。
- -G=3 模式：打开泛型的模式。

Go 1.18 中的 -G=0 模式和 Go 1.17 模式的比较显示，由于非泛型的变化，编译器的速度可能降低了~1%（因为 -G=0 模式不支持泛型）。

Go 1.18 的编译时间可能比 Go 1.17 慢 15-18%，这主要是由于实现泛型所带来的变化，也就是 Go1.18 开启泛型下，编译时间会变慢。

### 差异在哪

大部分的差异是由于新的编译器前端处理，因为 SSA 后端对于泛型完全没有变化。

- 在 -G=0 模式下（用于 Go 1.18 之前的所有编译器）：有一个语法分析器，创建 ir.Node 节点树的 noder 阶段，以及标准类型检查器。
- 在 -G=3 模式下：有相同的语法分析器，但程序首先由 types2（支持泛型）进行类型检查。

在通过 -G=3 模式打开泛型后，会有一个 noder2 阶段，使用语法信息和 types2 类型检查器的类型信息创建 ir.Node 节点树。在一次运行中，noder+ types1-typechecking 的开销总和约为 4%，而 types2-typechecker+noder2 的总和为 14%。

可以看到大部分的速度下降是由于改变了编译前端处理（并不意外）。

### 总结

可以明确的是，在打开泛型后，Go1.18 编译时间可能会慢 15-18%，Go 官方将计划在 Go 1.19 中减少这种额外的开销（计划，非明确）。在执行层面的时间开销，目前还没有过多的影响，这一块可以放心。

泛型的双刃剑初见，后续不管是编译时间、执行时间（预计不会减缓）、泛型的滥用、最佳实践等，都值得我们去讨论和关注。

