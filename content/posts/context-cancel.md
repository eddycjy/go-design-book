---
title: "增加标准库 Context 的取消 API"
type: docs
---

# 新提案：增加标准库 Context 的取消 API

大家好，我是煎鱼。

协程（Goroutine）是 Go 语言的一个大杀器，而我们常常需要在多个协程之间进行各种协调和通讯。


![图来自 @Soham Kamani](https://image.eddycjy.com/762cca33a78b2ea8084571653b03fb09.png)


所有 Go 有一个特别独特的东西，他就是上下文（context），你会在各种函数的第一个入参处见到他，标配了。

场景包含但不限于：
- 依赖 context 传递公共的上下文信息。
- 使用 goroutine 时进行异步操作，依赖 context 进行取消或返回错误等。
- 依赖 context 进行跨协程的管理和控制。

## 背景

标准库 Context 的 API 有许许多多种，今天的主角是 Cancel（取消）行为。

![](https://image.eddycjy.com/041679a9a4196a808be649cae2551660.png)

在代码中的 API 调用，如下：

```go
ctx, fn := context.WithCancel(ctx)
```

结合使用的案例来看，如下：


```go
func operation1(ctx context.Context) error {
	time.Sleep(100 * time.Millisecond)
	return errors.New("failed")
}

func operation2(ctx context.Context) {
	select {
	case <-time.After(500 * time.Millisecond):
		fmt.Println("done")
	case <-ctx.Done():
		fmt.Println("halted operation2")
	}
}

func main() {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)

	go func() {
		err := operation1(ctx)
		if err != nil {
			cancel()
		}
	}()

	operation2(ctx)
}
```

在上述程序中，当执行函数 `operation1` 后，假设返回了错误，就会执行 `context.cancel` 方法，将正在阻塞执行的 `operation2` 函数给结束掉。

这么来看，他是一个无比正常普通的 Go 程序。但这里有一个比较折腾的点，那就是你 cancel 取消了上下文后，只知道是被 cancel 了。原因是什么？

为什么被取消，没人知道...？

这就很苦恼了。我朋友在公司里经常看到这种案例，最后大家只能去翻日志或者根据蛛丝马迹去猜逻辑。

是比较不合理的。

## 新提案

在以前就有人提过类似 issues,就是想 “方便调试上下文被取消的地方”，也就是想解决被取消的场景处理。

经过几年的探讨后，@Sameer Ajmani 提出了新的提案《[proposal: context: add APIs for writing and reading cancelation cause](https://github.com/golang/go/issues/51365)》来解决这个问题。

会新增如下几个新的 API：

```go
package context

type CancelCauseFunc func(cause error)

func Cause(c Context) error

func WithDeadlineCause(parent Context, d time.Time, cause error) (Context, CancelFunc)

func WithTimeoutCause(parent Context, timeout time.Duration, cause error) (Context, CancelFunc)
```

使用案例：

```go
ctx, cancel := context.WithCancelCause(parent)
cancel(myError)

ctx.Err() // returns context.Canceled
context.Cause(ctx) // returns myError
```

在调用 `WithCancelCause` 或 `WithTimeoutCause` 方法后，会返回一个 `CancelCauseFunc`，而不是 `CancelFunc`。

其差异之处在于：可以通过传入对应的 Error 等类型的信息，然后在调用 
`Cause` 方法来获取其被取消的根因错误。

也就是既能得到被取消时的状态（context.Canceled），也能获取到对应的错误信息（myError），以此来解决前文中所提到的场景。

## 总结

这篇文章中，我们介绍了 Go 最常见的标准库 Context 的一个设计上的场景缺失。有 Context 状态码是不够的，在业务设计上，状态码和错误信息应该配套。

等该提案合并后，相信以往有过类似经历的同学，可以减少一定的排查时间了...