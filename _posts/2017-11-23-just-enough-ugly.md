---
layout: post
title: 恰如其分的丑陋
date: 2017-11-23
categories: tech
publish: true
---

用了段时间的golang，写点东西，强行把每年的post凑到3篇。

## The Good

### Language

做为C语系的明星级后辈，golang语法只能用平易近人来形容（跟lisp语系比起来）。只要是有一点其他C语系的背景，就算是没看完tutorial，不说写，读golang代码基本不会有任何困难。

*学习曲线平缓且短 check√*

作为2010年左右诞生的语言，modern的元素也一个都不差，reflection——有，GC——不断优化中，匿名函数——还行（不能用`=>`略不爽），类型推断——姑且算吧，etc。

*时尚 check√*

简洁。不仅仅是少了每行一个的`;`，或者是直接用方法名的大小写来区别“public”和“private”，就连语言的[Specification](https://golang.org/ref/spec)也只有java的1/N长。

*简约 check√*

我最爱的几个小地方：

```go
// swap
a, b = b, a
```

```go
// byte manipulation
bs := []byte("ffff")
fmt.Printf("%x", bs)
```

```go
// multiple expression for
for val, ok = dic[key]; ok {
    // val blahblah
}
```

*讨喜（？？？） check√*

### Go Tools

成熟的技术发展到一定阶段必定拥有自己的ecosystem，ecosystem的基石无非两点——开放&包管理。golang的开放不必说，全套代码就在那儿（compiler也用golang重写了，满足好奇心的成本够低了）。
golang的包管理也是简单的不行——源码分发，`go get`完事。对比js和py，npm和pip都是强大的包管理工具，从各方面来说都比`go get`强大不少（版本控制、可以做镜像等，现在的dep都还不是正式版本也比较神奇）。他们各自的生态体系中都诞生了不少不局限于nodejs和python的工具，比如[tldr](https://github.com/tldr-pages/tldr)和[thefuck](https://github.com/nvbn/thefuck)。
golang的第三方工具不见多少（当然你可以说是stdlib足够强大），而且略显混乱（顺便吐槽dep和godep竟然是两个不懂的东西），但人家第一方的工具厉害的没朋友啊：

#### pprof

CPU和heap都能dump，针对http也有特化的包支持。总之profiler该有的东西都有，而且：

![sample](assets/images/golang-heap-sample.png)

体贴到了这种程度！

BTW，CPU profile生成的SVG也不比Visual Studio里花哨的图差。

#### go vet

这段代码：

```go
if (a != b || a != c) && cond {}
```

是会有warning的！！因为`!=`是可以用xx定律提到外面去，而且代码本身的逻辑也读起来很不友好！是不是比`def but not used`之类的warning有用很多（golang中的`def but not used`是error）!

#### go fmt

终结“tab vs space”大战的神器，既然官方都认定了更好的格式，那还有什么理由不统一代码风格？不仅仅是一个项目的风格统一，而是**所有项目**的风格统一，想想都觉得很cooooooool（爸爸教你<del>玩游戏</del>写代码）。

google一下，吹gofmt的文章多得是。

### Language-Level Parallel

之前写过不少关于javascript的async的文章，但始终没有一个结论，业界也并没有找到银弹。究其原因，无论是`promise`还是`async/await`，始终只是**patch**，补丁打的再多，能有一件新衣服好看？

问题不是actor模式或者“actress”模式多么好，而是原生级别/语法级别的并行支持，直接就决定了程序员写代码的思路！就像用C-like的语言处理集合，你第一反应是`for`，而lisp-like，第一反应会是`fold`

有个形而上的[视频](https://blog.golang.org/concurrency-is-not-parallelism)值得一看

## The Bad

### no-sugar

爱好甜食，少了语法糖，多少有点不爽，比如

```go
// 存疑，因为golang并不是OOP，而且this并不总是指针
func Struct.SomeFunc() {
    this.Foo
}
```

```go
strukt := SomeStruct {
    A, // 同名的field自动赋值，like es2015
    B,
    C,
}
```

### no-functional

无穷无尽的for让人心烦，没有`map`没有`filter`，也没法写出“underscore.js”来。因为虽然有high-order function，但是没有泛型。既然golang的author出于效率原因并没有选择泛型，那你除了写for也没有其他选择。golang的设计思路应该是productive和effective的平衡，泛型这个特性明显是取effective而舍productive的决定。但设计思路这种东西其实还是挺玄的，“符合设计思路”这件事，其实就是“作者想怎么定就怎么定”，所以才会有很多大神A用B语言不爽愤而发明C语言的故事。/摊手

![why golang](https://cdn-images-1.medium.com/max/1600/1*nlpYI256BR71xMBWd1nlfg.png)

### error handling

显式的err handling，同泛型的取舍，有道理但我不喜欢，C式的错误处理让人感觉活在80年代。

## The Ugly

终于可以点题了。

虽然在大神眼里，golang只算是个[中等偏下](http://www.yinwang.org/blog-cn/2014/04/18/golang)的语言。
但这份“中等偏下”，IMHO，算得上是恰如其分。

拿另一个我用的多的语言举例子<del>当反面教材</del>——C#，C#每个版本里加入的语言特性和stdlib之多，简直是“互联网”式的。
C#好用吗？当然，你能想到的paradigm基本都能玩得转！但是C#2.0和C#6.0简直是两个语言，你写的C#和我写的C#也是两个语言。灵活的语言容易讨喜，写起来心情愉悦，殊不知“写代码一时爽，重构火葬场”。
你永远无法想象，在用别人的代码的时候，遇到`ins.变量1`这种代码心里会是怎么个感受。（真人真事，人格担保）

Python这么多年来把定位相似的Ruby按在地上摩擦，就是因为`Do one thing in one way`而不是`Do one thing in 93244 ways`，golang同理，简洁的语法和gofmt，很好的完成了`one way`的目标。比起python，golang更是多了效率（performance）的优势和原生的并行支持，备受追捧也是理所应当。

都2017年了，离C语言诞生都40多年了，可以说，最近10年诞生的所有语言都是DSL（domain specific）而没有general purpose了。
因为现在IT领域跟其他传统行业类似，也进入了细分的时代，比如AI和UI，虽然都有个I，但是从主流的开发语言到开发工具，基本上是没有走向统一的那天了。
就算一个新语言的定位是replace of C（Rust？不懂不乱说），那也是针对系统级编程的DSL。

**golang就是云编程的DSL**，因为docker和kubernets这种云时代的基础都是golang写的，正如unix是用C写的一样。

曾几何时，还被一些hello-world的文章误人子弟过，说什么golang是新一代的C，你家的C内存不透明反而封装好了线程池？

最后，还是一个我大学以来就坚持的观点——编程语言火不火，主要还是看用的人多不多，其他？都是废话。
