---
layout: post
title: Reading Lua (0)
date: 2015-09-10
categories: tech
publish: true
---

跟去年的[Learning Lisp](/tech/2014/07/30/learning-lisp.html)遥相呼应，作为今年自我KPI完成的总结文来写，这次逼格高点，第一次在Blog写个series。

计划的Content Table如下：

1.  [准备工作](/tech/2014/09/11/reading-lua-1.html)
2.  代码结构
3.  Lua有AST么
4.  此VM非彼VM
5.  GC
6.  TODO

还有一些泛泛的感悟，放在这一篇抒发比较合适：

Lua的源代码写的太DIAO了！Lua的源代码写的太DIAO了！Lua的源代码写的太DIAO了！重要的事情说三遍。跟原来读了好多次都没读下去的Ruby源代码不一样，一共就50+的文件和20000W左右注释详细结构清晰的CodeBase，给人心里压力小太多了。虽然语言的复杂程度不是一个量级的，但麻雀虽小五脏俱全，Lua作为一门完美实现了设计初衷的语言，代码肯定是同样值得一读的。    
Lua的实现没有任何依赖，而是用的是纯的Ansi-C！终于不用在Mac上make了，终于在Windows上也能out-of-the-box的跑起来了，对我们这种Windows低端程序员来说——太！开！心！了！     
指针对我们这种低端程序员来说还是好难，虽然终究能看懂，但是花时间略久。Macro虽然写的很花哨，但我会手动inline，所以也还好。