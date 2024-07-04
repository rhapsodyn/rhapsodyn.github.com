---
layout: post
title: 想写个编程语言，从哪儿开始下手？
date: 2024-07-04
categories: tech
---

尝试开始写短点，多写点，类似#浴室沉思#也不嫌弃。

--------

想写个编程语言，从哪儿开始下手？速答：打开编辑器就下手，[写就对了](https://github.com/rhapsodyn/vas)。

--------

如果不是想太`helloworld`的话，那做个编程语言，我理解就是两个**基本**需求：

1. 能用
1. 好用

能用：基础需求，一个语言要满足普通的*编程需求*，两点就够了：

1. [图灵完备](https://en.wikipedia.org/wiki/Turing_completeness)，化简一下：得有 `for` 和 `if`， 能读写`变量`，有简单的`操作`(+-*/)
1. 能做一些 `syscall`，化简一下：先得有个 `printf` 吧。

好用：`nice-to-have`需求，比如 `function` (`subroutine`) 、`while`、更多的`sprintf`、`operators` 等等。
至于为啥又是 `nice-to-have` 又是基本需求？因为没这些功能，很多测试用例你自己都不想写。

--------

一些关键的 `component` 的实现 (in my option)：

1. `tokenizer` && `lexer`: 用库也行，手写也不复杂，其实就一个状态机，一点一点吃源码吐`token`。
1. `parser`: `recursive descent parser` 还是很值得手写一次的，然后就是处理 `expression` 的各种经典算法了 （`pratt parsing` 等）。
1. `execution` (`runtime`): 可以直接当成 scripting 就解释执行了。有点追求就 emitting bytecode，当下最流行的就是编译成 wasm 了。最正经的肯定是用 LLVM backend，不过可能会搞错重点，想深入了解 LLVM 可以考虑。

