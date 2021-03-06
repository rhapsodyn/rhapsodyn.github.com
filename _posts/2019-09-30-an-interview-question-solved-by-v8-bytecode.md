---
layout: post
title: 性感V8字节码，在线教你解面试题
date: 2019-09-30
categories: tech
---

题目如下：

```javascript
{
  function a() {}
  a = 42;
}
console.log(a);
```

```javascript
{
  a = 42;
  function a() {}
}
console.log(a);
```

---

答案：

第一段代码输出`function`，第二段`42`

---

答案解析：

本题考察点在于第二段代码中的`a = 42`就只是一句赋值语句，并不是考生猜想的全局变量声明+赋值，而`function a`函数升舱以后，一直在最头部声明，也就是最后 log 出来的对象，而两句赋值，一个赋的是”局部的“，一个赋的是”全局的“。

吐槽：为什么一个语言的”全局变量声明+赋值“会和”赋值“语句长得一模一样？连有`goto`的 C 都不是这样啊

对吐槽的吐槽：不晓得为什么各种半灌水程序员（比如我），会热衷于对一个不晓得为什么就火了的三个星期就设计出来的语言吐槽，哪里来的自信（三个星期，可能也就够我把 parser 的测试用例写完……）

---

答案解析的解析：

### 1. 怎么看 JS 的 bytecode

首先，JavaScript 的 runtime 就很多，而有些实现根本就没有 bytecode 这种 IR(IL 随便了，都是一个意思)。忽略那些比较小众的 runtime（和 V8 的某些版本），怎么看各种实现了 IR 的 js runtime 的 bytecode，激发起了我一个编程语言爱好者强烈的兴趣。

“工欲善其事必先利其器”，如果能找到个 C#世界里 IlSpy 之类傻瓜的工具，事情不就——并没有找到（动手写一个？再说<del>=不说了</del>）

google 一圈以后，发现 node 自己就支持`--print-bytecode`这个 flag（实际上就是透传给了 V8)。**但是**，尝试之后发现就算只是`1+1`，最后 print 出来的东西都太多了，根本没法看。这个容易理解，毕竟 node 除了 libuv 这个核心，一大堆东西都是在 JavaScript 层实现的（比如`module`、`process`这种看起来 native 的函数）。

node 不行，可能就只有裸 V8 了。幸好还是有些比裸 V8 稍微好用一点的工具的：[eshot](https://github.com/bterlson/eshost-cli)，配合上[jsvu](https://github.com/GoogleChromeLabs/jsvu)，甚至连 QuickJS 和 Chakra 之类的也能一并看了。

eshost 的使用很简单，看看 README 就懂。我本机只配好了 V8 环境和 `--print-bytecode` 这个额外参数，实验够用了。

开始解题：

### 2. 这道面试题的 bytecode

先看赋值 42 在前的 bytecode：

```
[generated bytecode for function:  (0x07424ba1dd79 <SharedFunctionInfo>)]
Parameter count 1
Register count 4
Frame size 32
         0x7424ba1df26 @    0 : 12 00             LdaConstant [0]
         0x7424ba1df28 @    2 : 26 fa             Star r1
         0x7424ba1df2a @    4 : 0b                LdaZero
         0x7424ba1df2b @    5 : 26 f9             Star r2
         0x7424ba1df2d @    7 : 27 fe f8          Mov <closure>, r3
         0x7424ba1df30 @   10 : 61 2d 01 fa 03    CallRuntime [DeclareGlobals], r1-r3
    0 E> 0x7424ba1df35 @   15 : a7                StackCheck
    7 S> 0x7424ba1df36 @   16 : 81 01 00 00       CreateClosure [1], [0], #0
         0x7424ba1df3a @   20 : 15 02 02          StaGlobal [2], [2]
         0x7424ba1df3d @   23 : 26 fb             Star r0
   21 S> 0x7424ba1df3f @   25 : ab                Return
Constant pool (size = 3)
0x7424ba1dec9: [FixedArray] in OldSpace
 - map: 0x07426bc80799 <Map>
 - length: 3
           0: 0x07424ba1ddb9 <FixedArray[4]>
           1: 0x07424ba1de79 <SharedFunctionInfo gc>
           2: 0x07424ba1dd19 <String[#2]: gc>
Handler Table (size = 0)
[generated bytecode for function:  (0x07424ba1fca1 <SharedFunctionInfo>)]
Parameter count 1
Register count 5
Frame size 40
         0x7424ba1fde6 @    0 : 12 00             LdaConstant [0]
         0x7424ba1fde8 @    2 : 26 f9             Star r2
         0x7424ba1fdea @    4 : 0b                LdaZero
         0x7424ba1fdeb @    5 : 26 f8             Star r3
         0x7424ba1fded @    7 : 27 fe f7          Mov <closure>, r4
         0x7424ba1fdf0 @   10 : 61 2d 01 f9 03    CallRuntime [DeclareGlobals], r2-r4
         0x7424ba1fdf5 @   15 : a7                StackCheck
         0x7424ba1fdf6 @   16 : 81 01 00 00       CreateClosure [1], [0], #0
         0x7424ba1fdfa @   20 : 26 fa             Star r1
         0x7424ba1fdfc @   22 : 0c 2a             LdaSmi [42]
         0x7424ba1fdfe @   24 : 26 fa             Star r1
         0x7424ba1fe00 @   26 : 15 02 02          StaGlobal [2], [2]
         0x7424ba1fe03 @   29 : 13 03 04          LdaGlobal [3], [4]
         0x7424ba1fe06 @   32 : 26 f8             Star r3
         0x7424ba1fe08 @   34 : 29 f8 04          LdaNamedPropertyNoFeedback r3, [4]
         0x7424ba1fe0b @   37 : 26 f9             Star r2
         0x7424ba1fe0d @   39 : 13 02 00          LdaGlobal [2], [0]
         0x7424ba1fe10 @   42 : 26 f7             Star r4
         0x7424ba1fe12 @   44 : 5f f9 f8 02       CallNoFeedback r2, r3-r4
         0x7424ba1fe16 @   48 : 26 fb             Star r0
         0x7424ba1fe18 @   50 : ab                Return
Constant pool (size = 5)
0x7424ba1fd79: [FixedArray] in OldSpace
 - map: 0x07426bc80799 <Map>
 - length: 5
           0: 0x07424ba1fce1 <FixedArray[4]>
           1: 0x07424ba1fd11 <SharedFunctionInfo a>
           2: 0x0742a218e999 <String[#1]: a>
           3: 0x0742fe20c681 <String[#7]: console>
           4: 0x0742fe20c721 <String[#3]: log>
Handler Table (size = 0)
```

嗯，熟悉的`MOV`，熟悉的味道，跟想象中（和其他语言的）IL 还是比较一致的。bytecode 的各种知识不是本文重点，想要了解推荐阅读 lua 源代码（怀念在另一个平行时空坚持写完了的 lua 源码阅读系列文章）。而一些关于 V8 的基础知识[这个文章](https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775)写的很好了，看完也能了解个大概。

比较蛋疼的是 V8 并没有一个专门的页面列出所有指令和大概的用法，虽然根据命名也能猜到个大概，但是猜测哪是我们知识分子做学问的态度？本着科学精神，又是一番 google，发现 V8 的源代码里的[注释写的倒是挺详细](https://github.com/v8/v8/blob/4b9b23521e6fd42373ebbcb20ebe03bf445494f9/src/interpreter/interpreter-generator.cc)，作为指令简介列表应该是够了。（困惑的是 master 分支上的代码没有这个分支上的全，很多指令在 master 的 HEAD 上并不能搜到，但我的科学精神只能支持我到此为止了）

继续试验，发现两个版本的 IL 基本一致，除了下面几行：

```
# a = 42 在前
0x7424ba1fdfc @   22 : 0c 2a             LdaSmi [42]
0x7424ba1fdfe @   24 : 26 fa             Star r1
0x7424ba1fe00 @   26 : 15 02 02          StaGlobal [2], [2]
0x7424ba1fe03 @   29 : 13 03 04          LdaGlobal [3], [4]
```

```
# a = 42 在后
0x2fae7b71fdfc @   22 : 15 02 02          StaGlobal [2], [2]
0x2fae7b71fdff @   25 : 0c 2a             LdaSmi [42]
0x2fae7b71fe01 @   27 : 26 fa             Star r1
0x2fae7b71fe03 @   29 : 13 03 04          LdaGlobal [3], [4]
```

嗯，通过对照 V8 源码里的注释，大概看懂了就是把 42 赋值到了 accumulator，然后又赋给了某个 constant pool 的值？接下来还是一脸问号，还是没法说出为啥最后的输出是那个结果，犹如走进了迷宫完全不知道从何走起？这就对了，因为你还没有掌握走迷宫的终极大发——“从出口开始走”法。

我现在就带你从出口开始走，先看 console.log 到底在输出啥，整段代码就只有一次函数调用，如果你连`CallNoFeedback`就是对应的 bytecode 都猜不到，那确实可以放弃一切“走迷宫”的游戏了：

```
0x7424ba1fe12 @   44 : 5f f9 f8 02       CallNoFeedback r2, r3-r4
```

2 号 register 里（r2）放的是函数`log`，而 r3 是`this (console)`可以忽略，那么最后输出的 a 就是 r4 了，继续看 r4 里是啥：

```
0x7424ba1fe0d @   39 : 13 02 00          LdaGlobal [2], [0]
0x7424ba1fe10 @   42 : 26 f7             Star r4
```

嗯，是 constant pool 里的 2 号位（通过 accumulor 做了一次中转），那 constant pool 里的 2 号位是谁？一个是 42：

```
# a = 42 在前
0x7424ba1fdfc @   22 : 0c 2a             LdaSmi [42]
0x7424ba1fdfe @   24 : 26 fa             Star r1
0x7424ba1fe00 @   26 : 15 02 02          StaGlobal [2], [2]
```

另一个是某个 closure (function a）：

```
0x2fae7b71fdf6 @   16 : 81 01 00 00       CreateClosure [1], [0], #0
0x2fae7b71fdfa @   20 : 26 fa             Star r1
0x2fae7b71fdfc @   22 : 15 02 02          StaGlobal [2], [2]
```

好，跟我们的答案以及答案的解析完全一致，恭喜你走出迷宫了。

### 3. 面试题通用解法——翻答案

好了，上面就是翻答案的方法了，现在所有面试题都能解了，前提是你能带上你的笔记本<del>那直接跑一下代码不就得了</del>
