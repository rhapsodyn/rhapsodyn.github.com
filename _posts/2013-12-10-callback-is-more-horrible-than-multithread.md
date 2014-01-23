---
layout: post
title: 回调猛于多线程
date: 2013-12-10
categories: tech
published: true
---

写的js库提供了三个API——A、B、C，单独运行的时候好好的，但是当

{% highlight javascript %}
API.A();
API.B();
API.C();
{% endhighlight %}

的时候，灾难就降临了。我对这个library的设计的本意是所有接口都是声明式（Declarative programming）的，但现在任意调换上面三句代码的顺序，行为竟然不一样了。更扯的是，就算是用同样是顺序调用，产生的行为也会不一样。听起来很像是多线程是吧？

<!--more-->

问题出在——三个API可能会对同一个dom元素上的同一个event注册三个handler，而其中的某两个handler中有类似

{% highlight javascript %}
dom.on("event", function(target, event){
	var that = this;
	if(ready){
		doSth();
	}
	else{
		$.get.then(function(){
			ready = true;
			that.trigger("event");
		});
	}
});
{% endhighlight %}

的代码，天下大乱！

回调/异步纠缠在了一起，产生了多线程一般时空错乱的效果。好在js毕竟是单线程，不存在race-condition那种真正"无解"的难题。重构了一下代码，多注册了几个custom-event，多写了几个状态变量，虽然代码变得复杂无比，但好在问题还是解决了。

[Callbacks as our Generations' Go To Statement](http://tirania.org/blog/archive/2013/Aug-15.html)，感觉第二次引用这个链接了。

###update

第二天思路又清晰了点，感觉更好的实现方式应该是无论`API.A()`还是`API.C()`都应该只是enable some flags，然后event handler始终有且只有一个：

{% highlight javascript %}
dom.on("event", function(target, event){
	if(config.needToCallA)
		_A();
	if(config.needToCallB)
		_B();
	if(config.needToCallC)
		_B();
});
{% endhighlight %}

虽然可能这一个handler的逻辑会很重(extract to method AKA: ctrl+R ctrl+M)，但是至少是清晰的。