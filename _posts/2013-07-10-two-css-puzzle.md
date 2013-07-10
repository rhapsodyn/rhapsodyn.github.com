---
layout: post
date: 2013-07-10
title: "两个CSS的坑"
categories: tech
---

突然又觉得说是坑有点太过了，定位为trick应该比较合适

### hasLayout ###

[这个文章](http://www.satzansatz.de/cssd/onhavinglayout.html)已经写的相当的清楚明了了，我就只简单吐槽一下：这么奇葩的事情也只有M$干的出来了(想了一下，写的是*M$*而不是*M$的程序员*，因为换我的话，可能也会干出类似的事情吧……)

解决奇葩的问题，用奇葩的hack最好了(以毒攻毒)：
{% highlight css %}
zoom : 1
{% endhighlight %}

没事黑黑M$，开心一整天


### width: 100% - 100px ###

假设前提是不能使用[calc](http://caniuse.com/#search=calc)

{% highlight html %}
<div style="width: 400px; height: 400px">
	<div style="height: 40px"></div>
	<div id="target">
	</div>
</div>
{% endhighlight %}

现在想要让\#target填满父容器剩下的所有空间，而又不能使用`calc(100% - 40px)`，怎么办？

+ 直接写`height: 360px`？ 那如果父容器大小不确定怎么办？
+ 用JS计算？ 你赢了

实际上，我能想到的最优的解决方案是：

{% highlight html %}
<div style="width: 400px; height: 400px; position: relative;">
	<div style="height: 40px"></div>
	<div id="target" style="position: absolute; top: 40px; bottom: 0px;width: 100%;">
	</div>
</div>
{% endhighlight %}

最满意的地方在于——这根本就不是个hack嘛