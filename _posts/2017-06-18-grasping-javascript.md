---
layout: post
title: 熟练掌握javascript
date: 2017-06-18
categories: tech
publish: true
---

超过半年没有写文章，也是懒得很。

这么大个时间上的断层，免不了想总结回顾一下：

- 代码：这段时间写的最多的还是Unity和javascript，当然除了写还有学，写到哪儿学到哪儿（托了没那么忙的福，可以想看就花个一天看到底）
- 代码：`helloworld('elixir')` `helloworld('go')`
- 代码：最终还是入了vim的邪教，看个网页都想jjjjjj
- 看书：fiction和non-fiction一半一半，non-fiction把两本[简史](https://book.douban.com/subject/25985021/)看了，小说看了一堆，感觉越看越快
- 其他：积累管理经验？？（/nickyoungface）

好了，列完list就算是自我安慰完了（时光没虚度），接下来说正题

--------------------------------------

刺激到我想写文章的原因有两个：一是见识了个“写代码就是要用vim并且不能有任何提示所有方法都要自己背下来才是写得好”的技术总监，不以为然；二是看了[You Are Not Google](https://blog.bradfieldcs.com/you-are-not-google-84912cf44afb)这篇文章，深以为然。

技术（不单止computer science）的存在都是为了**解决问题**，解决问题的先决条件在于**正确识别问题**。

写代码（单指输入源代码）要解决的首要问题是能否默写出方法名么？并不是，而是如何更快更简单地用代码描述出你的思路！背诵方法名并不能辅助我思考，反而是completion能帮助我更快的输入，留出更多时间给思考。也让我在`10dd`（删除10行）了以后能心情好点，不受再次背诵之苦。

使用某个特定技术要为了解决某个特定的问题，你的问题（可能是只有3个程序员，需要做一个不知道有没有用户的APP试水）和Google的问题（有世界上最好的工程师团队，需要以毫秒级的响应速度完成数千万用户的请求）相距甚远，有多大概率你们问题的答案却刚好是同一个？

正确的识别问题，接下来，才**更有可能**完美的解决问题。

举个例子——javascript

--------------------------------------

这里说的javascript，不是指language层面的js，language层面的js连它的creator都承认过在前几个版本就是失败的存在（个人认为es4及以前的版本都是渣）。不过话又说回来，有哪个大家广泛认可的主流语言能在一个月里设计出来的？deadline才是产品最大的敌人！

runtime层面的javascript，确实是一个正确识别问题并解决问题的成功案例。要注意的是，这里的runtime并不单指V8，而是V8加上跟V8配套的browser或者nodejs的实现。就像同样也称得上成功的Unity，完整的runtime应该是mono加上底层更为复杂的render和control flow的实现。

各个runtime的实现可能有差异，但核心原理却大同小异，我们从用V8引擎的nodejs的intro开始：

> Node.js uses an event-driven, non-blocking I/O model

### Single-thread

虽然intro里的关键字是event-driven和non-blocking，但第一个keyword，还应该是单线程。

单线程要解决的是什么问题？答：复杂性。

就像前面说到的，V8只负责javscript的执行，且**只是以单线程执行javascript**。在完整的Runtime中，V8还需要跟其他功能协同作业，比如网络、比如绘制。如果javascript的执行环境就是个复杂的多线程模型，那么整个Runtime的复杂度将会是`M * N`而不是`M + N`，直接呈指数级增长。而且选择单线程模型，还有个额外的好处（也可能是设计者觉得最大的好处）—— 不容易犯错，毕竟普通意义上的多线程编程，跟正常人类的思维逻辑就有些相悖，并不是每个程序员都能驾驭的。（君不见现在的新语言都会把多线程用coroutine或者是actor之类的模型包装起来，不直接暴露给使用者）。

解决了复杂性的问题，V8的职责倒是单一明了了，那需要parallel的场景怎么办？

异步！

### Event-driven & non-blocking

这两个关键字加在一起，共同支持了javascript的并行，我们先从non-blocking说起。

看一个blocking的反例：

```
while(1) {
    block();
    render();
}
```

如果`block`一直执行不完或者是花了很长时间才执行完，那么`render`就会一直等待，你的浏览器可能就会出现“掉帧延迟”甚至无法响应各种鼠标键盘的输入，因为在`render`下面，可能还有响应各种事件和中断的函数在。

如何直接让任意blocking的方法变成non-blocking的，简单：

```
// 这段代码本身是错误的，并没有解决问题
non-block = function() {
    setTimeout(block, 0);
}
```

Javascript具有functional programming的特性，理论上所有blocking的函数都可以当成`callback`传给其他切换异步的方法。

OK，那是不是有了`setTimeout`是不是就万事大吉了？启个操作系统的timer，到时间了把该调用的callback都调一下就完事？

你忘了还有一类天天都能遇到的代码：

```
foo.on('click', clickCb);
```

除了各种浏览器event，还有network相关的地方，也需要回调的存在来共同完成异步这件事。怎么统一管理这些回调？

这就需要event-queue了（event-driven的核心）

直接从nodejs的网站上抄一段来，看下nodejs的event-queue都处理了些啥（once again：event-queue是V8之外的功能）

> timers: this phase executes callbacks scheduled by setTimeout() and setInterval().    
> I/O callbacks: executes almost all callbacks with the exception of close callbacks, the ones scheduled by timers, and setImmediate().    
> idle, prepare: only used internally.    
> poll: retrieve new I/O events; node will block here when appropriate.    
> check: setImmediate() callbacks are invoked here.    
> close callbacks: e.g. socket.on('close', ...).    

可以看到，event-queue为了解决异步模型，做了远比你想象多的事情。

简单来说，一个完整的event-loop分成了不同的phrase，每个phrase都是一个小队列，塞入了各种不同类型的回调，让V8执行回调，然后dequeue，当小队列为空或者达到threshold（`callstack.length == max`）,再处理下面的phrase，直至走完所有phrase，完成一次loop。

这些复杂性，对于V8或者使用javascript的程序员来讲，都是黑盒子，但却很好的解决了异步的问题（event queue + non-blocking callback），从而进一步解决了并行的问题。让你的浏览器能够“一边”执行js代码、“一边”渲染、“一边”进行网络IO。

### Compilation

单线程是好，但是从本质上，他无法利用多核的优势（虽然runtime的其他功能可以充分利用多核，nodejs也有cluster模式），可能代码的执行会不够“快”。

那要怎么解决这个问题？那就让他更快！

javascript编译器/VM给出的完整的答案十分的复杂，从对象模型到内存分配再到垃圾回收，无所不用其极。我能力有限，只取两点来讲：

#### no bytecode

我自己有很长一段时间好奇过，为什么从来不见有文章写javascript的bytecode的？因为在我的认知中，所有有VM和JIT的语言，比如C#/Ruby/Python，都该有bytecode，要不然怎么做JIT，怎么做进一步的优化呢？

这就是典型没想清楚问题的例子。

别忘了Javascript诞生之初，是为了给浏览器使用的。

有bytecode的执行过程是怎样的？

```
       parse      parse           JIT
source -----> AST -----> bytecode ----> executable
```

没有bytecode呢？

```
       parse      JIT              
source -----> AST ---->  executable
```

少了一个parse的步骤，在代码少的时候（早些年的js可不是动不动就好几十K，好几万个function哦），最后执行的总时间只快不慢，别忘了，javascript最开始可是以文本的方式下载到本地的。而且，javascript早年间都是很有大概率“一次加载一次执行”的，而不像是Ruby/Python的代码“一次加载多次执行”，缓存一份bytecode，就只用一次，实际上就是一种浪费。

但是，早些年是早些年，不巧现在不光浏览器的javascript越来越复杂，还出现了nodejs这种“幺蛾子”。

这种情况下，还能不更快？

#### hot optimazation

当然可以，如果nodejs在干跟RoR一样的事情了，那把bytecode再捡回来不就好了？

不过V8还是没有传统意义上的bytecode。因为所谓bytecode，跟LLVM的IR一样，其实就是多了一个中间地带来方便做一些“动态”的优化。V8也会做动态的优化，但并不是特别需要额外的特别地带，只需要一个从javascript function到executable的mapping就行。实际上V8会多开一个runtime profiler的线程，监视代码里运行次数最多（hottest）的函数，然后重点优化这些函数，直接替换最开始（第一次明显还需要有一个全量的编译，先让代码跑通，才能找到跑的最多的地方啊）生成的executable。

full compilation + hot optimazation 就是V8面对怎么才能让javascript跑的更快的答案。

------------------------------------

恩，以上就是我觉得能在简历上写 *熟练掌握javascript* 应该达到的水平。

------------------------------------

TL;DR : 先问，再答，是道。