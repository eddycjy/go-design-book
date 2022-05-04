---
title: "新提案：让 nil 不等于 nil，变成正确的"
type: docs
---

# 新提案：让 nil 不等于 nil，变成正确的

大家好，我是煎鱼。

各位有没有想过一个问题，就是 nil 有没有可能不是 nil？这在 Go 中还真的切切实实的发生了...

今天就由煎鱼带大家一起细细品尝。

## 背景

我们一起来看看如下代码：

```go
func main() {
	x := reflect.ValueOf(nil)
	fmt.Println(x.IsNil())
}
```

你觉得输出结果是什么，是 true 吗，还是 false？

不不不，运行结果如下：

```
panic: reflect: call of reflect.Value.IsNil on zero Value

goroutine 1 [running]:
reflect.Value.IsNil(...)
        ...
```

程序抛出了异常（panic），告诉我，你在 nil 值上调用 IsNil 方法会出问题...

这可太疯狂了，他说 nil 没法调用 IsNil，那这个方法存在的意义是什么？

## 新提案

实际上这是 Go 语言之父 Rob Pike 所提出的提案《[proposal: reflect: reflect.ValueOf(nil).IsNil() panics; let's make it return true](https://github.com/golang/go/issues/51649 "proposal: reflect: reflect.ValueOf(nil).IsNil() panics; let's make it return true")》里发现的 “奇怪” 案例。

如下图：

![](https://image.eddycjy.com/ec75471021670f18fef1c6d6ed2abe27.png)

可能就有同学问了？那什么情况下会是正确显示的，设想的使用案例是：

```go
type s interface{}

func main() {
	var v s
	elem := reflect.ValueOf(&v).Elem()
	println(elem.IsNil())
}
```
运行结果是 `true`，能够正确识别。

整体情况就是：nil 不等于 nil，这与我们的传统理解相差甚远，就像 interface 的相比一样折腾开发者的直觉。

原因没有太多的探讨，但这基本是设计上的 “缺陷” 了，当时没有考虑到这一种场景，导致出现了这个问题。

## 探讨

Go 核心团队的数名成员都进行了探讨，也是比较认同的。Roger Peppe 甚至给出了一个调整思路，如下：

- `reflect.ValueOf(nil)` 返回 Value 的零值（就像现在这样）。
- 对于 Value 的零值，IsNil 返回 true。
- 对于 Value 的零值，Type 返回 nil。

一般来说，nil Type 代表 "无类型"，就像零 Value 代表 "无值 "一样。而 nil 是我们在 Go 中得到的最接近于代表 "无 "的通用方法，所以 IsNil 返回 true 应该是比较合理的。

由于这个变动，许多开发者担心会违反 Go1 的兼容性保证，因此目前还在研讨阶段，需要等待最新的消息。

## 总结

今天这篇文章我们讲解了除了 interface 对比外，另外一个神奇的 nil 不等于 nil 的现象。在 Go 中，这些案例其实不少，大多被发现的也在源代码上标注了注释（但不是每个人都会去翻源代码的）。

同时一些设计上的漏洞就会由于 Go1 的兼容性保证一直遗留下来，积累许多年后，数量就会非常多。未来 Go2 真的能及时都处理掉吗？

这可能会比较玄学了。