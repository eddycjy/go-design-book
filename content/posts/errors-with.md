---
title: "新提案：新增 Go 错误处理中的套娃方法"
type: docs
---

大家好，我是煎鱼。

在 Go 的编程中，错误处理机制的处理永远是大家在讨论。不过 Go1 没法大动干戈了，那就想办法继续优化吧。

今天煎鱼给大家介绍一个五一假期期间学习时看到的一个新提案。

![](https://image.eddycjy.com/09ca09665f7f177e5796631ae4add710.png)


## 背景

在现阶段，我们在标准库中能够包装错误的唯一方法是使用 `fmt.Errorf`。

这意味着我们对错误所能做的就是将错误内容添加到其 `.Error()` 输出中，以此实现 error 类型。

如下代码：

```go
	err := fmt.Errorf("煎鱼：%s", errors.New("放假中"))
	if err != nil {}
```

如果这个时候，我们在收到错误信息时，想要返回堆栈并提供其他信息时，没有什么特别简单的方法。

只有如下 3 个选择：
- 你可以返回一些其他的错误，这就失去了原始错误的上下文。
- 你可以用 `fmt.Errorf` 来包装错误，这只是增加了文本输出，因此不是调用者可以通过编程检查的东西。
- 你可以写一个复杂的错误包装结构，其中包含你想检查的元数据，以此使用 `error.Is`、`.As`和 `.Unwrap` 来工作，以允许调用者访问错误的根本原因。

现在最靠谱的是第 3 种方式，最完整，对应的是在 Go1.13 新增的 error 系列方法，还在青壮年阶段。

确实原作者认为不够简单方便。

## 新提案

新提案是希望在标准库 errors 中实现一个更简单的函数来达到上述第 3 点的效果，支持将任何错误与任何其他错误包装在一起，从而使它们形成一个新的包装错误列表。

如下代码：


```go
// With returns an error that wraps err with other.  
func With(err, other error) error
```

被包裹起来的错误类似于链表，可以复用 `errors.Unwrap` 来遍历列表。而类链表存储，就有先后顺序的问题。

在 `With` 函数中，`other` 参数的错误将会放在包装错误列表的头部。如果在调用  `With` 函数时是 `With(b->a, d->c)`，呈现在内的错误列表是：d->c->b->a。

对应的使用场景：
- errors.Is(errors.With(err, other))：
  - 判别标准：errors.Is(other) || errors.Is(err)。
- errors.As(errors.With(err, other), target)： 
  - 判别标准：errors.As(other, target) || errors.As(err, target)
- errors.With(err, other).Error()：
  - 输出结果是 other.Error() + ": " + err.Error()。

提案作者@Nate Finch 希望通过这种错误包装方式，对既有的代码改动是最小的。也能提供最广泛的功能适用性，认为是有价值的。

## 案例

### 场景

作者给出了一个非常经典的用户案例。在我们平时写应用代码时，在写过的每个 go 应用程序中都看到了它。

应用中有一个返回特定域错误的包，例如返回 pq.ErrNoRows 的 postgres 驱动程序。

您希望将该错误向上传递到堆栈以维护原始错误的上下文，但您不希望调用者必须知道 postgres 错误才能知道如何从存储层处理此错误。

### 改造

可以使用新的 With 函数，您可以通过众所周知的错误类型添加元数据，以便可以一致地检查您的函数返回的错误，而不管底层实现如何。

如下代码：

```go
// SetUserName sets the name of the user with the given id. This method returns 
// flags.NotFound if the user isn't found or flags.Conflict if a user with that
// name already exists. 
func (st *Storage) SetUserName(id uuid.UUID, name string) error {
    err := st.db.SetUser(id, "name="+name)
    if errors.Is(err, pq.ErrNoRows) {
       return nil, errors.With(err, flags.NotFound)
    }
    var pqErr *pq.Error
    if errors.As(err, &pqErr) && pqErr.Constraint == "unique_user_name" {
        return errors.With(err, flags.Conflict)
    }
    if err != nil {
       // some other unknown error
       return fmt.Errorf("error setting name on user with id %v: %w", err) 
    }
    return nil
}
```

这种错误通常称为哨兵错误。

## 总结

今天给大家介绍的这个提案，还是比较贴合我们日常工作中的使用场景的。平时写 Go 应用程序，思考的多，就会折腾这个问题。会出现，莫非要根据错误文本来判断错误内容？

因此像是业内错误库，或是之前看毛老师讲的，都会进行相关的设计。这份提案也是一个不错的补充了。


## 参考
- [提案内容来自《proposal: errors: Add With(err, other error) error》](https://github.com/golang/go/issues/52607)