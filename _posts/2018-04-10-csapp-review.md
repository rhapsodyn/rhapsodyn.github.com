---
layout: post
title: cs:app读书笔记
date: 2018-04-10
categories: tech
publish: true
---

突然有时间可以看下这本砖头，就挑着想看的章节，先看个一遍。越看越感叹先人的智慧，计算机科学当得起科学二字，毫不夸大。

### c 语言

CS 的精髓就在于抽象，把抽象说的具体一点：汇编就是 executable 里.text 的抽象，而 C 就是汇编的抽象。为什么很多 system 级别的事情只能 C 来做，原因很简单：隔着两层抽象（还有一层是虚拟机之类的东西），你很难做到跟 C 一样的事情。
`longjmp`怎么做？用`try-catch`吗？自由度开到顶的指针怎么做？用蹩脚的`unsafe`吗？（参考 Microsoft 自己开源的 C#的 codebase，就算是偏下层的产品，用过多少`unsafe`）
“支持”和“为此而生”是两个截然不同的东西，就像 SUV 和真正的越野车。

linux 的 syscall 为啥都是以 C 提供的，跟这本书的所有实例都是用 C 写的，一个道理。写不好 C 说得过去，不能读 C 万万不行。

### 进程

“进程是操作系统提供给开发人员的一种抽象，程序看上去独占了机器的所有资源”，这个定义，感觉比我们本科教材上写的好。（好像写的是：进程独占资源，线程占用处理能力？）

### %rax %eax %ax %al

联想到 tyr 大神公众号的推送文章，“现在的程序员”无脑就用 json+http，实在是太浪费了。人家一个 64 位的寄存器都可以分 32 位、16 位的用，你凭啥把一个 bit 就能解决的 bool，用`false`这么五大个 ascii（更不要说 utf 了），5 \* 8 = 40 个 bit 才搞定？绿水青山金山银山了解一下？

### x86-64 加了一条限制，传送指令的两个操作数都不能指向内存位置

TIL 系列，虽然在日常编程中都把内存当“高速缓存”在用了，但是在某个维度上来看，内存确实是龟速。只是很多程序员都没法活在那个维度。

### branch prediction

最早看到这个概念应该还是在某个 StackOverflow 的回答里，简单（不是很正确）的描述就是，处理器为了做 eager 的优化，会先忽略`if`的结果，先“并行”计算`if{}`里面的结果，如果猜对了，皆大欢喜，直接就有结果了；猜错了，那就再算一次，多多少少浪费了一点性能（其实跟你预期的时间一致）。

也就是：

```
if (cond) {
	doSth
}
```

处理器会先预测一个 cond，然后可能会预先 doSth，哪怕最后`cond==false`（点背）。

有个优化的方法：用类似`cmovge`之类的指令（也就是`?:`），来规避原先`if`编译出的`jge`指令，断绝了 cpu 猜测的想念。

前两个月很火的 meltdown 还是 spectre 漏洞中的某一个，就是用这个原理，去读到内核的地址里的东西的。

### 栈破坏检测

又是类似`0xDEEDBEEF`之类的烂梗，只是多科普了一个知识：金丝雀检测之所以叫金丝雀检测，是因为以前下矿井会先下只金丝雀，因为它能察觉有毒的气体。

### 库打桩机制

又是个 TIL：C 提供某种能力，可以在编译时替换第三方库的函数为自己的实现，从某种程度上来说，很像 ruby 里的 Monkey Patch，但是要彻底的多。

### 动态链接库

直接节约了内存中的 .text .data 空间，再次体现出系统级的省；热升级——nginx 不停，后台代码更新。这两点是我没有想过的。

### fork

写了些 fork 的 hello world，深刻理解了朋友说的：fork 以后的子进程直接共享 parent 的状态和内存，你告诉我，windows 怎么做？但要写更复杂的 fork，心智上的负担就重了，还是类似于 coroutine 之类简单一点的模型比较符合我等的智力水平。

### virtual memory

* 分级（多级的Page Table Entry）和缓存（硬件级别的TLB），简直是CS中万物理论（中的一项）。
* 需要通用的数据结构，加个header总是没错的。系统分配的heap内存还有一个可选的footer