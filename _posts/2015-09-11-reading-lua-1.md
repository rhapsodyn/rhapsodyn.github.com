---
layout: post
title: Reading Lua (1) 准备工作
date: 2015-09-11
categories: tech
publish: true
---

如果说后面几篇对lua源码的分析是干货的话，这一篇应该可以说是“废话”了。但事实上，看完了这篇，后面几篇你都不用看了。    
因为这一篇跟后面几篇的关系——简单来说就是——渔和鱼。     
在这一篇，我只写**自己**是怎样读代码的，而不提具体的代码。之所以把**自己**\<b\>了，是想要强调此处的渔是非常主观和个人化的东西，必须不适合所有人，就权当自己的经验总结和采坑大全了。

<!--more-->

想要来一顿美味的Lua源码阅读大餐，你需要准备以下食材：

1.  一个Visual Studio (不是说GDB不好，只是在断点的时候直接就转到上下文中函数的定义或实现，这种体验谁用谁知道)
2.  一份原汁原味的Lua5.1.4源代码（这个版本可参考的资料最多，虽然Lua各种总的loc都不多，但简单而完备的版本看起来肯定更爽）
3.  各种参考资料适量（云风的GC系列文章、ANoFrillsIntroToLua51VMInstructions、Mike Paul在各种地方留下的阅读指引）
4.  足量的耐心
5.  足量的时间

有了以上几种食材，烹饪的过程就异常简单了——F5！

没错，是他，是他，就是他：

![F5](/assets/images/F5.jpg)

跟广为流传的MikePaul推荐的[Reading Order](https://www.reddit.com/comments/63hth/ask_reddit_which_oss_codebases_out_there_are_so/c02pxbp)不同的是，我觉得还是一上来就F5靠谱的多。默念函数名也是读F10也是读，running code总是比plain code更生动易懂（还有一种可能是我水平太low，大神都是可以直接通读脑中构建running state的上下文的）。当然，所谓“F5读代码大法”，并不是直接在`main`的第一行打上一个断点以后就不停的`F10`和`F11`就行了。而是要在了解源代码大体结构（下一篇具体来写）的基础上：

1.  确定一个feature
2.  根据初读代码和猜测（怎么猜？靠函数名和变量名呗），在大致的位置打上断点
3.  构造Lua代码
4.  观察程序是不是会跑到这个断点
5.  验证feature，整理整个流程

For example: 我想研究Lua的GC，看文章，大致Scan代码，发现GC是分好几个步骤的比如`GCSsweepstring`或`GCSsweep`，那么当垃圾不包含`string`的时候，在一次GC里还会执行`GCSsweepstring`这个步骤吗？

构造lua代码如下：

{% highlight lua %}
local foo = {}
for i=1,1000 do
	foo[i/10] = { a = i, b = i+1 }
end
{% endhighlight %}

断点当然是打在`lgc.c`的这个位置：

{% highlight c %}
	case GCSsweepstring: {
		lu_mem old = g->totalbytes;
		sweepwholelist(L, &g->strt.hash[g->sweepstrgc++]);
{% endhighlight %}

然后，F5走起。到断点了么？Callstack是什么样的？LuaState相关的field都是怎样的？解答自己的疑惑，再不停发问，继续解答。如此递归，就离这道大餐上桌的时间不远了。

一些PS：

*  VS中的Compile as C Code或者Command Argument怎么设置，google一下就知道
*  宏没法打断点，实在搞不懂里面的实现就只能手动inline了，记得ctrl+z回来就好，虽然不改回来也没什么大问题
*  接上一条，大胆的改代码做实验写注释，弄坏重新下一份源代码就好