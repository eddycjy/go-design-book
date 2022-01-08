---
title: "泛型历史"
type: docs
---

# 泛型历史

在 2020~2021 年，Go 语言的泛型讨论频频出现在各微信群，且冲上了国内外各大文章的 “头条”。

如下图：

![来自 p 神公众号的截图](https://image.eddycjy.com/c9c48e9479c7036f7d5a33b6ab49e855.jpg)

好家伙，我最早了解到时是考虑 Go1.16 释出，后面又推到了 Go1.17，接着现在又延期到了 Go1.18 了（2021 年底）。按规划，Go1.18 将正式释出泛型，终于不鸽了。

看到了信息的表象后，再想想为什么泛型 “这件事情” 突然醒目起来了，其原因之一是由官方 [Go，11 岁](https://blog.golang.org/11years) 的博文所引爆的，做了一轮宣传。

同时近日举办的 GopherCon2020 大会，Robert Griesemer 分享的 Typing [Generic] Go。更正式的让 Go 泛型更面向了大众，也侧面的说明官方认为其已经到达了一个新的阶段了，进入最终实现阶段。

事不宜迟，既然官方都已经摩拳擦掌了，我们的学习之路也得跟上，本文将会介绍 Go 泛型是什么，并了解他的过去和历史，

## 什么是泛型

泛型程序设计（generic programming）是程序设计语言的一种风格或范式。泛型允许程序员在强类型语言中编写代码时，使用一些以后才确定的类型，其在真正实例化时才会为这些参数指确定类型。另外各语言和其编译器、运行环境对泛型的支持均不一样，因此需要针对来辩证。

简单来讲，泛型就是参数化多态。其可根据实参类型生成不同的版本，支持任意数量的调用：

```
func F(a, b T) T{ return a+b }

// T 为 int
F(1, 2)

// T 为 string
F("1", "2")
```

在编译时期编译器便确定其 T 的入参类型。这也是 Go 泛型实现的要求之一 “编译时类型安全”。

## 为什么需要泛型

这时候可能会有人说，没有泛型也可以啊...感觉写业务代码没什么影响，与其搞泛型不如搞好 errors（具体可见：错误处理章节）。

但泛型是有其所需的场景，最常见的是像基础库在处理获取配置中心数据时，就要处理类型，时常遇到下述场景：

![image](https://image.eddycjy.com/4d630c956a58bd4b88a4a6e0cddbb845.gif)

如果使用接口（interface）类型来做，也得 `switch.(type)` 枚举出所有的基础类型。这显然并不合理，也没法做太复杂的逻辑，而且所支持的类型还泄露。

另外同时单从语言层面来讲，泛型支持是一个必然事件了，因为泛型的存在对解决特定领域的问题存在一定的意义。没有的话，总是差了一块领域。

## 接口和泛型有什么区别

在上面我们有提到接口（interface）类型，这时候就出现了泛型的第二个经典问题。那就是 “接口和泛型有什么区别？”，为什么不用接口来实现 “泛型”：

```
type T interface { ... }
func F(a, b T) T { return a+b }
```

也像这么一回事，但在这里存在一个致命的缺陷。那就是接口的入参和出参均可以在运行时表现为不同的类型：

```
F("煎鱼", 233)
```

要做好，还得依靠内部去对参数进行断言，否则作为 string 类型的煎鱼又如何和 int 类型的 233 相加呢，那是必然报错的。

而反过来看真 “泛型” 的实际使用，编译器会保证泛型函数的入参和出参必须为同一类型，有强制性的检验：

```
// 报错：type checking failed for main
F("煎鱼", 233)

// 必须为同一类型，才能正常运行
F(666, 233)
```

两者存在本质上的区别，泛型会更安全，能够保证编译早期就发现错误，而不是等到运行时（并且可能会存在隐性的 BUG）。

总体来讲，泛型相较接口有如下优点：

- 更安全：编译早期就能发现错误。

- 性能好：静态类型。

核心思想的转变：**在没有泛型前，接口是方法集。在有泛型后，接口是类型集**。

## 为什么那么久都没有泛型

前几段在社区的微信群看到一位小伙伴吐槽 “Go 语言居然没有泛型？”，变相来看，可能其会认为 ”Go 都已经 11 岁了，2020 年了居然还没有泛型？”。

这显然是不对的，因为泛型本质上并不是绝对的必需品，更不是 Go 语言的早期目标，因此在过往的发展阶段没有过多重视这一点，而是把精力放在了其他 feature 上。

### 泛型的发展历程

另外 Go 语言在以往其实进行过大量的泛型 proposal 试验，基本时间线（via @changkun）如下：

|  简述   | 时间  | 作者 |
|  ----  | ----  | --- |
| [Type Functions]  | 2010年 | Ian Lance Taylor |
| Generalized Types  | 2011年 | Ian Lance Taylor |
| Generalized Types v2  | 2013年 | Ian Lance Taylor |
| Type Parameters  | 2013年 | Ian Lance Taylor |
| go:generate  | 2014年 | Rob Pike |
| First Class Types  | 2015年 | Bryan C.Mills |
| Contracts  | 2018年 | Ian Lance Taylor, Robert Griesemer |
| Contracts  | 2019年 | Ian Lance Taylor, Robert Griesemer  |
| Redundancy in Contracts(2019)'s Design  | 2019年 | Ian Lance Taylor, Robert Griesemer |
| Constrained Type Parameters(2020, v1)  | 2020年 | Ian Lance Taylor, Robert Griesemer |
| Constrained Type Parameters(2020, v2)  | 2020年 | Ian Lance Taylor, Robert Griesemer |
| Constrained Type Parameters(2020, v3)  | 2020年 | Ian Lance Taylor, Robert Griesemer |

虽然偶有中断，但仔细一看，2010 年就尝试过，现在 2020 年了，也是很励志了，显然官方也是在寻路和尝试的过程中，但一直没有找到相较好的方案，争端过多了。

直至 2021 年，正式确定了 Go 泛型的内容和方向，这将在下一节《泛型设计》具体展开介绍和说明。

### 为什么纠结那么久

Go 团队认为要将泛型的概念融入 Go，并与系统的其他部分很好地配合，必须解决一些深层次的技术问题，而在当时并没有解决这些问题的办法，所以拖了这么多年，一直在不断地尝试。

关于这些问题，在几年前就在博客上写过一篇《[The Generic Dilemma](https://research.swtch.com/generic "The Generic Dilemma")》：

![](https://image.eddycjy.com/8ce3c37629e8051e08e66a3143fce9c6.png)

即使克服了那一页上的问题，也有其他问题，接下来你会遇到的问题是：”如何让程序员以一种有用的、易于解释的方式省略类型注释“。也就是如何更人性、更易于的表达泛型的类型参数。

在写该文的现在（Go1.18），已经释出了泛型的正式提案和代码实现。引来了大量的同学吐槽，表示 “眼花缭乱、太过于丑陋、学习成本高” 等，Go 官方担心的果然成真了。

### 与其他语言交流

Go 团队和一些真正的 Java 泛型专家谈过，他们每个人都说了大致相同的话：要非常小心，它不像看起来那么容易，而且你会被你犯的所有错误困住。

一个 Java 示范，可以浏览一下《[Java Generics FAQs - Frequently Asked Questions](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html "Java Generics FAQs - Frequently Asked Questions")》的大部分内容：

![](https://image.eddycjy.com/29573ea87fc02ecc0215f01b16993580.png)

看看过了多久你会开始思考 "这真的是最好的方法吗？"。

在泛型过程中会遇到许多问题，像是《[How do I decrypt Enum<E extends Enum<E>>](http://www.angelikalanger.com/GenericsFAQ/FAQSections/TypeParameters.html#FAQ106" "How do I decrypt Enum<E extends Enum<E>>")》：

![](https://image.eddycjy.com/b3d6ea4704175a81e91762ec9953e163.png)

为此，Go 团队在泛型的推动上非常谨慎。


## Go 泛型尝鲜

泛型尝鲜的方式有两种方式。线上 Ian Lance Taylor 提供了一个在线编译的 [go2go](https://go2goplay.golang.org/)：

![image](https://image.eddycjy.com/0609310f0a775b57fe017f56c1e50195.jpg)

另外一种是线下，也就在本地安装 Go 的特定分支版本（若是在 Go1.18 以后，直接使用 Go 最新版本就好了）：

```
$ git clone https://github.com/golang/go
$ git checkout dev.go2go
$ cd src && ./all.bash
```

不过这种本地安装的方法会耗时比较久，初步尝试的话建议使用 go2go 就可以了。而在尝鲜时，可以看到在代码块中声明了一个 `Print` 方法，其函数签名主体分为三部分：

![image](https://image.eddycjy.com/5fc715bb226563645dfc6bb4da210c84.jpg)

咋一看，变量 T 的这个关键字 `any` 是什么？早期泛型你可能有听说合约（Contract），难道这就是合约。其实严格意义上来讲并不是，因为为了更一步简化语法，合约在 2020.06.07 已经正式移除。

其已改头换面，现在只需要写参数化的 interface。而上述的 `any` 关键字是一个预定义的类型约束，声明后将允许任何类型用作类型实参，并且允许函数使用用于任何类型的操作。

从语法分析的角度来讲，`Print` 方法一共包含了如下属性（从左到右）：

- type list：声明了入参的类型列表为一个 `T` 变量，其可以传任意类型的参数。

- parameter list：声明了入参的参数列表为 `T` 变量的切片，且形参为 `s`。

- return type list：声明了函数的返回参数列表。

上述函数签名便是一个 Go 泛型的基本样子，由于本文并不是 CRUD 泛型，便不展开案例，若大家有兴趣可以详细阅读提案：[Type Parameters - Draft Design](https://github.com/golang/proposal/blob/master/design/go2draft-type-parameters.md)。

## 泛型的战争

### 为什么不用尖括号

在社区中很多同学在讨论的一个问题，那就是 “为什么 Go 泛型不像 C++ 和 Java 那样使用尖括号？，也出现了 “Go 一直标榜业界工程实践类的榜样，为什么就是不用尖括号” 的言论？

思考问题我们不只看表面，官方说不行，那么我们可以倒推来看，看看 Go 语言就用尖括号：

```
func print<type T>(list []T) {

print<int>(numbers)
print<string>(strings)
print<float64>(floats)
```

普通的函数声明看上去似乎结构清晰，没有什么大问题的。接着往下看：

```
a := w < x
b := y > (z)
```

我们继续把代码演进一下，简洁一点：

```
a, b := w < x, y > (z)
```

这时候就犯难了，不仅编译器难以解析，人也很难判别，到底指的是：

```
a := w < x
b := y > (z)
```

又或是：

```
a, b := w<x, y>(z)
```

从上述代码来看，使用尖括号难以分别，因为没有类型信息，就无法确定赋值的右侧是一对表达式 `w < x和y > (z)`，还是返回两个结果值 `w<x, y>(z)` 的泛型函数实例化和调用，其存在歧义。

要解决还要引入新的约束，会破坏 Go1 的兼容性承诺，这显然是不合理的。

### 为什么不用括号

其实最早 Go 泛型的版本是使用了括号的模式，虽然能用，但是用括号会引入新的解析歧义。例如：

```
var f func(x(T))
```

从语法上来讲，你无法识别他是未命名参数的 `x(T)` 函数，还是类型名为参数的 `(T)` 函数。同时 Go 语言还存在强制类型转换这一语法，假设代码是 `[]T(v1)` 和 `[]T(v2){}` ，那么你在开括号处，就无法得知其是否代表类型转换。

更甚至在函数的完整声明上，我们都会感到困惑：

```
func F(T any)(v T)(r1, r2 T)
```

函数入参、泛型、返回值声明均都是括号，造成了语义不清，这显然也是不合理的。

### 为什么不用书名号（«»）

想的美，Go 官方并不想使用非 ASCII，未来更没打算支持。

## 总结

在本文中我们从多个维度介绍了 Go 泛型的相关内容，既了解到了上段时间 Go 泛型再度火爆的信息来源是什么。也知道了 Go 泛型是什么，与接口的区别。

同时我们还针对业界常见的一些疑问，例如接口和泛型的区别，泛型的历史，泛型的尖括号/括号/书名号之争进行了解释和说明。

很显然，在保证 Go1 向后兼容性的同时，Go 官方也不想直接妥协出一个随便的方案，因此总是不断地在改进。随着 Go 语言的不断应用，泛型也和 errors 一样被推上风头浪尖。

最终在 2022 年的 Go1.18 版本释出了泛型设计，并不断地完善，实现标准库等的配套支持。

## 参考

- [Type Parameters - Draft Design](https://github.com/golang/proposal/blob/master/design/go2draft-type-parameters.md)