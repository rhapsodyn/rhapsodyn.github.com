---
layout: post
title: The hardest bug i've debugged
date: 2013-11-01
categories: tech
published: false
---

全部都是看这个帖子有感[What's the hardest bug you've debugged?](http://www.reddit.com/r/programming/comments/1pk14s/dave_baggetts_answer_to_programming_interviews/)

先说文章本身——只有四个字——不明觉利，我的感觉是这样，作者自己也是这种感觉。（我就不吐槽最后那个渣总结了，什么“用到了量子力学debug”，真心不知道怎么得出这个结论的）

然后发现程序员真心是一类奇怪的生物，什么东西都能拿来炫耀：最难的bug、最奇葩的注释、最难懂的代码etc，简直就是“伤疤是男人勋章”心理的升级版。

说回主题，这两年来我遇到的最厉害的bug——<del style="color:gray">因为海森堡测不准原理哥德巴赫猜想引起的</del>网卡罢工。

流水账模式switched√

某次版本发布，第一天风平浪静。第二天站点突然就挂掉了，而且直接是挂到ping不通了。嗯，机器出问题了，一定是这样，用另一块网卡远程reboot了机器，站点又跑起来了。过了一段时间，又挂了。突然世界观就有了崩塌的迹象——我用的是asp.net，离网卡还有两层远呢，再怎么在clr上写代码，也不能把操作系统写到crash吧。还是坚信是操作系统自己的问题，跟我的代码无关。运营人员把代码回滚了，整整两天没crash过了。

这下真是三观尽毁了。

开始认真的分析问题：1.从代码层面入手，找两个版本的diff 2.从log入手，不放过一切蛛丝马迹。

第一条路不太好走，因为两个版本之间的diff真心有点太多了，而且从心理和经验上，我都不认可有代码可以把系统搞crash