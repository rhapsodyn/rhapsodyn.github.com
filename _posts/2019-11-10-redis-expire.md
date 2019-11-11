---
layout: post
title: Redis Expire
date: 2019-11-10
categories: tech
---

其实本篇本来是一个还挺多`h3`的大纲，只是其他条目我已经无法想起来究竟有啥好写的了，就只简单写一个主题——redis 过期 key 的策略。

简单[翻译一下](https://redis.io/commands/expire#how-redis-expires-keys)，redis 有主动过期和被动过期两个策略：

被动过期：就是 lazy 的策略，当访问到某个 key 的时候，会先检查 expire，如果已经过期了，就回收这块内存。

主动过期：redis 以 1 秒 10 次的频率（redis expire 的精度只能设置到秒），实施以下三个步骤：

1. 随机找 20 个 key 和对应的 expire
1. 删除所有过期的 key
1. 如果第二步删除了 25%以上的 key（也就是 5 个以上），则重复第一步

看起来简单，却*相对*完美的解决了问题。

首先，问题（需求）是什么：

有过期时间的 key 需要有符合预期的行为——get 没过期的 key 时返回 value，get 过期的 key 返回 nil （当然还有可以手动 get/set expire)

如果只有主动过期策略，那么只有保证在**每个**最小时间周期上轮询**每一个**key 的过期时间，才能满足需求，这个过程在 key 多的情况下不可接受的，不仅仅是类似于 GC 的 stop the world，而且还只有挂起所有写操作，在频繁写的情况下根本就无法 restart。

那如果只有被动过期呢？stop the world 的问题解决了，但是在很多 key 写入以后不再读的情况下，内存又被用爆了（没有 read，就没有 delete）。

那势必只有两个策略的结合才行了，那么主动过期的周期如何设置？其实如果是一般人设计的系统，随便给个经验值已经可以算是个 90 分的解决方案了，不过 redis 毕竟是 redis，那个随机算法很好的保证了系统内**最多只有占写操作 25%的 key 该过期而又没被删除**，还是在可接受的性能代价下。

---

看下来这套解决方案跟 GC 的策略还挺像的，但是

1. GC 有一个难点和痛点是对象间的引用关系，比如循环引用问题就可以直接让简单的引用计数打出 GG，redis 没有这个问题
1. 在有 GC 的环境中，使用者不用关注资源何时释放（也是自动 GC 的意义所在）。而在 redis 中，使用者需要而且必须明确资源何时释放

这两个点决定了，过期策略和 GC 策略基本上是两个东西。

---

感慨时间：

1. redis mysql 这种老牌项目真是宝库，经久不衰有各种层面上的原因
1. timer 是个好东西，人人都需要它。游戏引擎里 [update](https://gameprogrammingpatterns.com/game-loop.html)，nodejs 之类 runtime 里的 [event loop](https://nodejs.org/de/docs/guides/event-loop-timers-and-nexttick/) 都是些翻答案才能发出“原来如此”赞叹的解决方案。不过多翻翻答案，也是一条快速涨分的好路啊
