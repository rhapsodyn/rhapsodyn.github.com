---
layout: post
title: Learning Lisp
date: 2014-07-30
categories: tech
publish: true
---

> (MEMBER \`TRY \`(DO DO-NOT))

QQ签名从`["do","do not"].include? "try"`改成了上面的，眼看今年给自己的KPI——*learning a language every year*完成在即，心情大好。感觉自己编程功力又上升了一个小段位，下半年剩下的时间好好啃下[SICP](http://www.amazon.cn/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A8%8B%E5%BA%8F%E7%9A%84%E6%9E%84%E9%80%A0%E5%92%8C%E8%A7%A3%E9%87%8A/dp/B0011AP7RY/ref=sr_1_1?ie=UTF8&qid=1406723686&sr=8-1&keywords=sicp) (看亚马逊评论说中文翻译的不太好，看来又只有原版走起了)，争取能给自己打个S。

Lisp，逼格最高的编程语言之一（比他逼格更高的没人出来炫耀，比他逼格低的那就太多了）。在没`TRY`过之前，我觉得我肯定也是`DO-NOT`了。好好看了下，发现对我们这些C语系出身的程序员来说，Lisp真心可以算的上是**mind-blowing**了。极简的规则（上面那个quote已经包含了Lisp语法中所有最基本的元素了），一切都是list，而一切又都是执行（evaluating）。道生一，这似乎在哲学上更容易生出万物，也更有道法自然的感觉。

对lisp理解还停留在感性层面，道的出的牛逼也只有上面这些了。

Talk is cheap, show me the code。我懂，这个[repo](https://github.com/rhapsodyn/SumAll)里有四个版本（Lisp Javascript Ruby C#）的代码，都只在干一件事情——计算一个多层嵌套的数组的和。先只贴Lisp的版本出来：

{% highlight cl %}

(DEFVAR *L* `(1 2 3 (1 (2 3))))

(DEFUN SUM-ALL (LIST)
       (COND ((ENDP LIST) 0)
             ((NUMBERP (FIRST LIST))
              		(+ (FIRST LIST) (SUM-ALL (REST LIST))))
             (T (+ (SUM-ALL (FIRST LIST)) (SUM-ALL (REST LIST))))))

(SUM-ALL *L*)

{% endhighlight %}

这段简单的递归能很好的体现出Lisp的表现力。平心而论，对于C#这种不支持异构数组的强类型语言来说，这对比似乎不怎么公平，因为C#版本里很大部分篇幅都用来构造数组了。而且递归这种东西好像本来就是从Lisp来的，写递归函数不就是在扬长避短么。对于你们这种怀疑我专门用特别的实例厚此薄彼的行为，我只想说，**泥垢了**。

再次总结一下，现阶段Lisp于我来说炫酷得没朋友的特性就两点：**极简**的定义&&**自顶向下**的表现力。

PS1:入门看的是德国某大学的Common Lisp的在线教程，整个课程的设计简直是良心，还有statistic：

![learn_lisp_statistic](/assets/images/learn_lisp_statistic.png)

PS2:Lisp的方言选择也是<del>装逼活</del>门艺术，以后就用Scheme了。我又不需要practical（我的武器库很牛逼，不要惹我），Common Lisp对我来说就太大了点。

SICP我来了。