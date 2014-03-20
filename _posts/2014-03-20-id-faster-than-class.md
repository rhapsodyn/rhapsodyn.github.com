---
layout: post
title: 无论如何，id还是比class快
date: 2014-03-20
categories: tech
publish: true
---

[http://jsperf.com/queryselector-id-vs-class](http://jsperf.com/queryselector-id-vs-class)

虽然这种结论用小脚趾头想也能得出来，还是jsperf了一下，92%的slower还是挺出乎意料的，基本上就跟sql有索引和没索引的差距一样大了（其实应该原理也类似才对）