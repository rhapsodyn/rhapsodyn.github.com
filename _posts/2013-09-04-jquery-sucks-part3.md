---
layout: post
title: JQuery Sucks (3) —— 失败的promise
date: 2013-09-04
categories: tech
---

崇拜这种高端黑：[http://domenic.me/2012/10/14/youre-missing-the-point-of-promises/](http://domenic.me/2012/10/14/youre-missing-the-point-of-promises/)

原本以为promise是jquery中为数不多的亮点，结果发现还是可以黑。有空了想写点demo来验证下上面那个文里的scenarios2-4是如何不成立的（或许1.10/2.0版本已经进化到可用了） 

BTW：咱就说[1.8之前的promise用起来别扭](/tech/2013/07/26/jquery-sucks-part1.html)，原来本来就是个坑啊。看来我最近的“认坑”能力有提高啊

BTW2：又发现了jquery之所以有这么多槽点的一大原因——没有“Do one thing”。如果jquery专注于dom manipulation，可能不会有现在这么多槽点。不过真只专注于这一点的话，jq那不就变成了sizzle加强版了么？没有现在“Do every thing”的功能，可能jquery也就没那么火了。还真矛盾。