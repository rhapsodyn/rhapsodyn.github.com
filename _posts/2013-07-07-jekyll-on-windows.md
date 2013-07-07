---
layout: post
date: 2013-07-07 21:10:00
title: "在windows上用jekyll太难了"
categories: tech
---

研究了一下午，结果发现跟所有linux产品的windows移植（or阉割）版一样：想要用jekyll，就免不了——**折腾**

整个折腾过程如下：
1. 安装jekyll（废话）
2. 一来就遇到个编码的问题，google了半天还是stackoverflow靠谱，环境变量里改一下就可以解决了：
	{% highlight powershell %}
set LC_ALL=en_US.UTF-8
	set LANG=en_US.UTF-8
	{% endhighlight %}
3. 为了格式化上面那段代码又产生了另一个蛋疼的问题：[jekyll依赖于python](https://github.com/mojombo/jekyll/issues/1182)
4. 然后就是最新的pygments竟然有[BUG](http://stackoverflow.com/questions/17364028/jekyll-on-windows-pygments-not-working)
5. 终于折腾完毕，终于可以在我大windows上调试了