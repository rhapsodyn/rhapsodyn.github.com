---
layout: post
title: My Tool Chain
date: 2015-03-06
categories: tech
publish: true
---

> timediff(now, lastpost).month \> 6

这半年其实收获良多，各种language和各种runtime加上各种platform玩来玩去，想写的能写的值得一写也挺多，不急，慢慢来，记着就好。

念念不忘，必有回响。<del>再拖拉我还是会写</del>

这篇的主题是ToolChain——也可以叫——当我重装Windows/Mac时我弄些什么。

### Alfred/Launchy

Alfred已经太有名了，无需多言。在windows上我能找到的（用过的）最好的替代品就是[Launchy](http://www.launchy.net/)了。  
实话实说，win7点了win键以后输入程序名相对xp来说已经有了很大提升了，但比起launchy这种专业的软件来说还是差了一大截。    
特别是可以自定义程序搜索路径和记住最近打开的程序（e.g：我现在alt+space | c就是conemu，alt+space | ch就是chrome，alt+space | u就是unity）这两个feature，真心好用。   
关于这种全局快捷键app的必要性，上古大神Alan Kay（可能记错）就说过：newbie用鼠标，frequent user用快键盘+鼠标，pro只用键盘。不用鼠标带来的效率<del>逼格</del>可以详询*nix over Windows党。    
最后再说一个Launchy的好：donate只要2$，对比起sublime的70$（我觉得sublime也很好，但是没有好到35倍 || 如果我的工资发的不是￥而是$我就买），就是买买买啊。

### Chrome

也算做过一段时间的web开发，各种浏览器出于工作也好个人喜好也罢，都用过一圈了，最后还是觉得Chrome最**顺手**（这类软件没有什么最好，只有最顺手）。    
好像原来的原来还写过firefox plugin的post，插件这玩意也就是锦上添花，firefox现在跟Chrome比已经算不上*锦*了。    
具体的有多*锦*就不表了，现在必须的*花*只有手势这一朵了（还是稍微一提：Chrome的帐号系统确实好用，至少不用像以前的firefox一样每次重装都折腾一次插件了）。    

### VisualStudio/Sublime

Visual Studio出了[community](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)版本以后，Windows上*最强*的IDE基本就没任何争议了，随着MS越来越拥抱开源，其在社区里的口碑也越来越好了，现在我黑MS的欲望都越来越弱了。    
Sublime作为Mac/Windows上都通用的Editor，对于我们这类小白开发（大神的Vi/Emacs都是比VS还厉害<del>难用</del>的多呢），着实是比其他文本编辑器好用了一大截。    
这两个共同的优点就是1.feature多 2.插件多 3.GUI功能也很好用。诚然用鼠标的效率比不上用键盘，但鼠标悬停显示各种变量怎么也比用`print`和`pp`让人愉快很多吧。

### Python/Ruby/Github

Python和Ruby优先级高于其他运行时（Mac预装了）的原因在于很多小工具都是基于这两个runtime的，其他lua lisp node什么的就看心情了。关于ruby gem有个小tips：

{% highlight powershell %}
gem sources --remove https://rubygems.org/    
gem sources -a https://ruby.taobao.org/    
gem sources -l
{% endhighlight %}

必须要赞一下taobao，在国内的开源界确实比其他公司走的要早要远。    
Github for Windows这个GUI我觉得比tortoise还要好用，for Mac就更不用说了，Mac能战的SVN GUI根本就没有。

### ConEnu

在Windows下——支持tab页+自定义font+支持ctrlc+ctrlv+最大化窗口的命令行工具——就我用过的——只此一家，所以没得选了。

### What's More

TODO
