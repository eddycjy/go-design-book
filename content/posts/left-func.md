---
title: "Go2 增加 “左侧函数” 来解决错误处理"
type: docs
---

# 新提案：Go2 增加 “左侧函数” 来解决错误处理

大家好，我是煎鱼。

错误处理一直是 Go 一个很有争议的地方，大家在该类提案上贡献了各种各样的想法。在五一假期期间，我也发现了一个有趣的技术提案，那就是：左侧函数；还有 Go+ 的新思路。

今天就由煎鱼带大家一起来看看。

## Go 新提案：左侧函数

在现有 Go1 的错误处理机制下，我们一般处理错误都需要写大量的 `if err != nil` 的逻辑。

有人笑称 100 行里有 50 行是以下代码：

```go
func main() {
 x, err := foo()
 if err != nil {
   // handle error
 }
 y, err := foo()
 if err != nil {
   // handle error
 }
 z, err := foo()
 if err != nil {
   // handle error
 }
 s, err := foo()
 if err != nil {
   // handle error
 }
}
```

于是在社区里有多位小伙伴就提出了**左侧函数**这种想法。

希望借此来解决错误处理的问题，减少每次多写的 3 行左右的代码，实现一致的错误处理方法。

涉及如下提案：

- 《[proposal: Go 2: errors: allow function on left hand side of assignment](https://github.com/golang/go/issues/52416 "proposal: Go 2: errors: allow function on left hand side of assignment")》
- 《[proposal: Alternate to try(): 1. Call func/closure from assignment and 2. break/continue/return more than one level](https://github.com/golang/go/issues/3247 "proposal: Alternate to try(): 1. Call func/closure from assignment and 2. break/continue/return more than one level")


提案中的新代码如下：

```go
fmt.Errof("%v, %w", a, err) := simple()
```

简化写法：

```go
errorHandle(err) = io.Copy(w, r)
```

新的处理思路，就是加一层（万能的软件架构处理方式），用左侧函数来处理所有的错误。

## Go+：错误表达式

与 Go 有关系的一员：Go+，也做出了自己的《[ErrWrap expressions](https://github.com/goplus/gop/wiki/Error-Handling "ErrWrap expressions")》错误处理方案，在前面的提案中有一定的人进行了讨论，因此大家可以一起评估看看。

### 表达式介绍

核心的思路是对错误处理增加了表达式的语法机制。如下：

```
expr! // panic if err
expr? // return if err
expr?:defval // use defval if err
```

下面我们一个个展开介绍。

表达式 `expr!` 检查 valN 是否为零。如果没有，它会恐慌。对应的 Go 代码：

```go
val1, val2, ..., valN1, valN := expr
if valN != nil {
    panic(errors.NewFrame(valN, ...))
}
val1, val2, ..., valN1 // value of `expr!`
```

表达式 `expr?` 检查 valN 是否为 nil，如果不是，它将返回错误。对应的 Go 代码：

```go
val1, val2, ..., valN1, valN := expr
if valN != nil {
    _ret_err = errors.NewFrame(valN, ...)
    return
}
val1, val2, ..., valN1 // value of `expr?`
```

表达式 `expr?:defval` 检查 valN 是否为 nil。如果不是，它使用 defval 作为expr的值。对应的 Go 代码：

```go
val1, val2 := expr
if val2 != nil {
    val1 = defval
}
val1 // value of `expr?:defval`
```

### 演示代码

具体的示例代码：

```go
import (
	"strconv"
)

func add(x, y string) (int, error) {
	return strconv.Atoi(x)? + strconv.Atoi(y)?, nil
}

func addSafe(x, y string) int {
	return strconv.Atoi(x)?:0 + strconv.Atoi(y)?:0
}

println(`add("100", "23"):`, add("100", "23")!)

sum, err := add("10", "abc")
println(`add("10", "abc"):`, sum, err)

println(`addSafe("10", "abc"):`, addSafe("10", "abc"))
```

输出结果：

```
add("100", "23"): 123
add("10", "abc"): 0 strconv.Atoi: parsing "abc": invalid syntax

===> errors stack:
main.add("10", "abc")
	/Users/xsw/goplus/tutorial/15-ErrWrap/err_wrap.gop:6 strconv.Atoi(y)?

addSafe("10", "abc"): 10
```

基于表达式进行错误处理的机制优化之余，还增加了错误堆栈的信息跟踪。

## 总结

今天这篇文章中，我们针对 Go 现在 “焦头烂额” 的错误处理机制的提案进行了讨论，前有 try-catch、panic 替代等，现有左侧函数、表达式等新的思路。

你觉得这几种错误处理方式怎么样呢，可以解决不？

欢迎在评论区和留言和交流。

### 阅读更多

- [先睹为快，Go2 Error 的挣扎之路](https://mp.weixin.qq.com/s/XILveKzh07BOQnqxYDKQsA)
- [我们对 Go 错误处理的 4 个误解？](https://mp.weixin.qq.com/s/Ey-yqIq__wpaLTlBAOHjxg)
