---
layout: post
title: Promise&&最具体原则&&sth-else
date: 2013-12-30
categories: tech
published: true
---

又是年底了，又到了各种有感而发的时候，每到这个时候，思绪都会乱到想写个文都只有起个“Promise&&最具体原则&&sth-else”这种奇葩名字了。

心态好一点，少点吐槽多点思考，才是，真的。

### Promise

早就想写点promise的东西了，不过一想到linus大神的“talk is cheap，show me the code”，决定还是要code先行。

开了个[repo](https://github.com/rhapsodyn/Spai)，自己一定要在1.1写完一个最简单的promise实现，要不然又要觉得13年白过了。

实现的具体细节和收获等写完了代码再写，先说promise——其实也没什么好说的——promise一定是js的未来（或者说是async handling的未来），promise + generator = c#里面的await（嗯，M$有时候确定挺厉害的，比如MVVM比如ajax比如很多东西），而且最近很多浏览器都有native的promise实现了。

### 最具体原则

其实这个名字是我自己起的，其存在的具体例证有二：

1.前段时间在win08r2上配IPSec的时候，有个具体案例：有两条rule存在，一条是`localhost:anyport to remote:anyport deny`另一条是`localhost:1233 to remote:1233 permit`。这两条规则不管是以什么顺序新建,本机到远程的1233端口永远都是开放的，也就是说第二条规则的优先级永远是大于第一条的。MSDN上给的解释是以“more specific”的那条为优先。

2.不管是$()还是querySelector，永远是写的越详细的规则跑的越快。比如，`$("div#id")`就要比`$("#id")`快。

分析：感觉还是缓存在作怪（起作用），凡是这种读多写少的场合都应该多多少少有一些类似缓存的机制。而用做缓存的hash一定会是有很多key指向同一个value这种情况的（最大限度的空间换时间），如果规则就是key的话，为啥是更具体的先命中？其实我也没想清楚……

应用：没想清楚还是能有些有用的结论——以后不管是使用API还是设计API，在遇到正则匹配或者是类似的东西的时候，一定是“the more specific， the better”的。

### sth-else

多思考，多积累，多跟人交流吧，sth懒得记了<del style="color: gray">其实是因为我搞忘了</del>。