---
title: "三元运算符"
type: docs
---

# 三元运算符

大家好，我是煎鱼。

这是一个很多其他语言工程师转 Go 语言的时间节点，这就难免不论一番比较。其中一个经典的运算上的就是 “三元运算符”：

![image](../../../images/why-not-operator.png)

为什么 Go 语言不支持三元运算符，Go 不支持三元运算符就是设计的不好，是历史在开倒车吗？

今天就由煎鱼来和大家一起摸索为什么。

## 三元运算符是什么

三元运算符，在典型的数学意义上，或者从解析器的角度来看，是一个需要三个参数的运算符。而我们日常中，最常见的是二元运算符：

```golang
x + y
x / y
x * y
```

还有一元运算符：

```golang
-a
~b
!c
```

以及今天的男主角 “三元运算符”。
在 C/C++ 等多种语言中，我们可以根据条件声明和初始化变量的习惯来选择性使用三元条件运算符：

```c
int index = val > 0 ? val : -val
```

## Go 使用三元运算符

想在 Go 语言里也使用三元运算符时，发现居然没有...想要实现与上面相同的代码段的方式似乎只能：

```golang
var index int

if val > 0 {
    index = val
} else {
    index = -val
}
```

看上去十分的冗余，不够简洁。

## 为什么 Go 没有三元运算符

为什么 Go 没有 `?:` 操作符，没有的话，官方推荐的方式是怎么样的。

通过 Go FAQ 我们可以得知：

![image](../../../images/go-faq-operator.png)

Go 官方就是推荐我们使用前面提到的方式来替代，并且明确了如下态度：

- Go 中没有 `?:` 的原因是语言的设计者看到这个操作经常被用来创建难以理解的复杂表达式。
- 在替代方案上，if-else 形式虽然较长，但无疑是更清晰的。一门语言只需要一个条件控制流结构。

Go 语言的设计者是为了考虑**可读性**拒绝了实现三元运算符，"less is more." 也是标榜台词了。


## 社区争议

Go 语言的一些点与众不同，基本是大家皆知的。无论是 if err != nil，又或是本次的三元运算符，要大家用 if-else 替代：

```golang
if expr {
    n = trueVal
} else {
    n = falseVal
}
```

### 反对和同意

#### 反对

因此有社区小伙伴给出了反对，基本分为如下几类：
1. 认为 if-else 也有以类似情况能被滥用，设计者的理由不够充分，认为是 “借口”。
2. 认为三元运算符的 “丑陋” 问题，是开发者的编码问题，而不是语言问题。三元在各种语言中很常见，它们是正常的，Go 语言要有。
同意
3. 认为用 if-else 替代三元运算符也很麻烦，让开发者多读了 3-4 行和额外的缩进级别。

#### 同意

认可这个决策的也有不少，为此给出了大量的真实工程案例。

一般来讲，我们用三元运算符是希望这么用：

```
cond ? true_value : false_value
```

你可能见过这么用：

```
cond ? value_a + value_b : value_c * value_d
```

还见过这样：

```
(((cond_a ? val_one) : cond_b) ? val_two) : val_three

cond_a ? (val_one : (cond_b ? (val_two : val_three)))
```

还能嵌套三元运算符：

```
int a = cond_a ? val_one :
    cond_b ? val_two :
    cond_c ? val_three : val_four;
```

也能出现可读性更差的：

```
void rgb_to_lightness_(
  const double re, const double gr, const double bl, double &li)
{
  li=((re < gr) ? ((gr < bl) ? bl : gr) : ((re < bl) ? bl : re) +
                            (gr < re)
                          ? ((bl < gr) ? bl : gr)
                          : ((bl < re) ? bl : re)) / 2.0;
}
```

说白了就是真实的代码工程中，大家见到过大量三元运算符滥用的场景，纷纷给出了大量的难理解的例子，让大家困扰不堪。

## 总结

在这篇文章中，首先针对 “三元运算符” 做了基本的介绍。紧接着根据 Go 语言不支持三元的态度进行了说明，且面向社区的争议我们分为了正反方面的基本诠释。

实际上一个简单的 `?:` 既整洁又实用，但是没有很好又高效的办法方法可以防止丑陋的嵌套，也就是排除可读性的问题。

在真实的业务工程中，常常能看到一个三元运算符，一开始只是很简单。后面嵌套越加越深，逻辑越写越复杂。从而带来了许多维护上的问题。

给大家抛出如下问题：

- 你认为 Go 语言是否要有三元运算符呢？
- 如果要有，复杂嵌套的三元运算符又如何考虑呢？

欢迎大家在评论区留言和交流 ：）


## 参考

- [What is the idiomatic Go equivalent of C's ternary operator?](https://stackoverflow.com/questions/19979178/what-is-the-idiomatic-go-equivalent-of-cs-ternary-operator)
- [What is the reasoning behind Go not having a ternary conditional operator?](https://www.reddit.com/r/golang/comments/5dqpab/what_is_the_reasoning_behind_go_not_having_a/)
- [Should Go have a Ternary Operator? Or was it left out intentionally?](https://www.reddit.com/r/golang/comments/4fsh1m/should_go_have_a_ternary_operator_or_was_it_left/)
- [We don't need a ternary operator](https://dev.to/mortoray/we-dont-need-a-ternary-operator-309n)