---
title: "Go 和 Golang 有什么关系"
type: docs
---

# Go 和 Golang 有什么关系

大家好，我是煎鱼。

写这一节时，掐指一算是招聘季了，无论是校招、社招、HR、面试官们都蠢蠢欲动。这不，我有一个朋友的 HR 朋友都有起名困难了，一看 Go 语言的工作说明（Job Description），发现各有不同。

如下图：

![来自某招聘网站](https://image.eddycjy.com/c45688ebcaf6be66de4944b32ddd0d47.png)

仔细一看，有叫 Go 的，也有叫 Golang，还有叫 GO 的。好家伙，Go 语言有这么多个别名，甚至某乎都讨论了起来。到底叫什么是正确的？

为此，今天就由煎鱼带大家理一理，了解这背后的关系。

## 官方定义

从网上的资料来看，大家对 Go 的名字还是比较关注的，对于 Go 团队来讲，仿佛经常被问。例如：

- “Go 和 Golang 的关系是什么？”
- “Go、Golang、GO 哪个对？”

甚至在之前探讨 Go2 草案时，也有人开始起 Go2 的名字了，纠结是要叫 “golang2”，还是 “go2lang”：

![](https://image.eddycjy.com/7e56599419b48eac630b16c1784ac5c8.png)

其实这是错误的。在 Go FAQ 中有明确的回答这个问题：

![](https://image.eddycjy.com/c4165189cd37a21c2aacec89db27fec3.png)

这一门语言称为 “Go”，不叫 “Golang”，也不叫 “GO”。“golang” 只是网站的地址，而不是语言的名称。

同时 “GO” 的语言名称叫法也是错误的，虽然官方上的 Logo 是 “GO”：

![](https://image.eddycjy.com/5b3cb8222310993b4b0bbd0e40fee714.png)

但这显然只是设计师层面的美观考量，并不是这一门语言的标准定义。

因此**这一门语言叫做 “Go” 语言**，这是正确的，也得到官方认证的，也不曾改变过。

## 为什么会有 Golang

但可能又有小伙伴疑惑了，那为什么 “Golang” 这个别名，如此之火。到底是为什么？

这里一共有三点原因，分别是：站点地址（Go FAQ 提到）、搜索引擎、社区和论坛、语言重名。

### Go 站点地址

Go 团队所期望的 https://go.org 早就被注册，从网站的底部标识来看，2008 年起建站：

![](https://image.eddycjy.com/619030e343831bb211e30a1594a21377.png)

所以 Go 语言只能使用 https://golang.org，你也会 https://pkg.go.dev 和 https://golang.org、https://godoc.org，存在多个域名，并不统一。

因此作为 Go 开发者所常用官方站点，自然而然 golang 这一个语言标识就深深地被记住了，一直沿用至今。

同时域名为 “golang” 关键字，自然会大幅度的影响到 Go 资料搜索引擎的收录，是一个非常重要的因素。

### 搜索引擎

在早年 Go 语言还不知名时，用 go 关键字去搜索资料会非常的困难。这是各大搜索引擎早年的一个槽点（reddit 很多吐槽）。

因为单一的 go 关键字过于广泛了，很多人会直接用 golang 关键字来搜资料，反而会更能看到一些与 Go 真正相关的。

![](https://image.eddycjy.com/f85c981423574a8bed81b2794c0a839f.png)

这一点在近年来有明确改善，得益于 Go 语言的崛起，现在也能搜到了。

### 社区和论坛

在社区、论坛等，也有类似的问题。因为占位、重名、认知等原因。像是 segmentfault、twitter 叫 golang。掘金叫 Go，各有不同。

![](https://image.eddycjy.com/9abcc275d41c5b65a7a031698c01ca7c.png)

这点难以改善，毕竟各家都是不同企业的。所以难受的点是用户，搜了 Go，可能搜不到，又跑去搜 Golang 才可以。

再看看国外的论坛，在 Google 群组 golang-nuts 和 golang-dev 也有类似偏差。

基本可以明确 **“Golang” 更多会被用在搜索和标签上**，能够保证搜索和标签查询的结果。

### 语言重名 

实际上在 Go 语言出现前，已经存在一门 “Go!” 的编程语言了。有网友表示这也是 Go 官方纠结的一点。

![](https://image.eddycjy.com/cf947f737d7a7b4b2ae97ad55811e71a.png)

不过实际上编程语言重名并不少见，但由于真实性有待考量，建议仅是了解即可。

至少现在已经没有这门语言的命名之争。

## 总结

可以明确，官方诠释的正确名称为 Go。

但由于 go.org 域名的原因，
因此在 Go Programming Language 的通俗称呼下，采取了 golang 来作为 Go 站点、Google 群组的域名/组别等的建立。

Go 资料肯定都集中在官方站点、论坛，自然而然，大家用 “go” 关键字也就很难搜索到了，都得用 “golang” 关键字。

可以明确，**Go 是这一门编程语言的名字，Golang 更多是在搜索和标签上的使用**。