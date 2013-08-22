---
layout: post
title: (function(){})()
date: 2013-08-21
categories: tech
---

黑JQuery的系列文章一直都还没写出来，今天读JQuery代码，反倒是多了点体会。（话说回来，JQuery的源代码可能是让我收益最多的JS代码了）

原来一直觉得`(function(){ /*do something*/ })();`就只在闭包的时候有点用。立即调用，把`function(){}`这个壳子去掉不就行了。

我只能说，我又一次图样图森破了。	
看这段代码：

{% highlight javascript %}
(function repeat() {
	//do something
	setInterval(repeat, 10);
})()
{% endhighlight%}

是不是要比下面这段代码更简洁也更有表现力？

{% highlight javascript %}
function repeat() {
	//do somthing
	setInterval(repeat, 10);
}
repeat();
{% endhighlight%}

还有一种情况，`(function(){})()`也能体现出表现力的优势，看下面两段代码

{% highlight javascript %}
//Setup
doA();
doB();
//SetupEnd
{% endhighlight%}
	
{% highlight javascript %}
(function setup(){
	doA();
	doB();
})();
{% endhighlight%}

后面那段代码好像只是把注释的地方变成了一个脱了裤子放屁还白白浪费巨（pi）多（ge）性（xing）能（neng）的函数调用。however，IMHO，注释太容易被忽略，能强有力的表达（搞不好还可以重用），why not？