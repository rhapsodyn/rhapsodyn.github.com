---
layout: post
title: Embedding Mono
date: 2014-07-23
categories: tech
publish: true
---

前后折腾了好几天，终于把Demo跑起来了，全部的代码见这个[repo](https://github.com/rhapsodyn/EmbeddingMonoDemo)，虽然行数不多，基本意思算是表达的比较清楚了（cpp里的注释都用心在写了，在我大Windows上跑起来的方法见上一篇[post](/tech/2014/07/17/why-unity3d-has-no-designer-cs.html)）。在天天用.NET的时期找了各种借口一直都没好好研究过hosting，这次用Mono做了个demo，算是把这个念想给圆了。

嗯，念念不忘，必有回响。

心得我只捡重要的，一个OL说完好了：

1.  学好C很重要，不看Linux <del>（实际上是看不懂）</del>，看个ruby|mono什么的还是需要的。还是觉得\#define什么的好难懂，想吐槽都不知道从何吐起。

2.  对比unity的MonoBehavior，觉得我的[Lib](https://github.com/rhapsodyn/EmbeddingMonoDemo/blob/master/lib.cs)的设计更合理啊！毕竟`virtual`不就是干这个的吗，而且，在子类写完`override`就能出智能提示也要比光靠记忆去写`Start()`更科学啊。unity这样设计，应该是出于简单的考虑（这性能的提升可以忽略不计吧）？

3.  在unmanaged的环境里去找method其实跟在managed环境里用反射做的事情和做的结果都挺相似的，当然，如果有很多假设（custom code的命名方式命名规则等等），就会少了很多读metadata的麻烦事儿。

4.  设计framework/library确实比application要难得多，抽象和设计的重要性/关键性是做application比不了的。

5.  开源就是好，你在想知道细节的时候可以去看，而不是猜。

6.  unity studio更像是photoshop而不是visual studio，没的辩。unity用mono做script engine也就是看上了它的跨平台而已，如果ruby python什么的跨平台做的足够好，可能也轮不到mono了。

做这个Demo感觉经验值涨了不少，不过还是要过两天用*build on mono*的形式写个一样功能的Demo，才算圆满。