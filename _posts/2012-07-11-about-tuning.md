---
layout: post
title: About Tuning
categories: tech
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
**--edit：**想了一下还是多加点东西写成个UL吧，正式开始工作也一年了，写个几千字的长文不太可能，就稍微多记点东西充数吧


+ 介于我还是太菜！心得只有一句话：你觉得perf没问题的地方——可能会有问题；你自己都觉得有问题的地方——一定有问题！
+ 我真的太尼玛菜了！数据库的字段浪费太严重了，各种nvarchar(200)起码浪费180
+ 彩笔我是！想了半年了，单元测试还是没写起来，到12月为止，再不写unittest就不买新PC了
+ 菜比！做一堆垃圾操作，明明是DB=&gt;DB就完事的，非要从DB=&gt;APP=&gt;APP=&gt;DB，还多select一堆不需要的column浪费带宽。不要求perf就不去管perf了吗？
+ 但是彩笔不会迷信大神！这段可能略长，我就再多一个p

大神经常会告诉凡人各种Best Practice，类似于写JS都用===不用==，记下来了并且常常实践肯定没错。但是如果不知道===是为了

1. 避免cast的消耗
2. 可能会有潜在的BUG的话，

那彩笔永远也变不了大神，永远也不能写Best Practice给人家看的。（我自己的BP就是——如果不是大循环，我都用==的，一来不熟JS的人看了我代码不会疑惑、至少减少了一次baidu的耗时，二来有时候我并不要求精确，我就是需要implicit cast）