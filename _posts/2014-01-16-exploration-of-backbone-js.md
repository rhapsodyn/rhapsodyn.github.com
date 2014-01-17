---
layout: post
title: Backbone.js初探
date: 2014-1-16
categories: tech
---

运气不错，有时间可以研究一些新玩意，就抓起了原来一直没好好看看的Backbone（严格来说，Backbone是现在流行的MV* javascript大军里资历最老的几个了，只是对我来说是新而已）

一边code一边看posts一边就开始写文章好了，前戏都省了，直接就是Pros/Cons:

## Pros
----------------------------------------------------------------
### Convention

“约定胜于配置”，这句话是好久以前（大四？）听老赵讲asp.net mvc1 preview的时候提到的asp.net mvc从RoR中汲取的优点之一。无论是Jeremy Ashkenas在u2b上某点击量最高的presentation中的某页ppt上，还是在Backbonejs的文档中，你都可以看到**Convention**这个词的出现。

以我浅薄之见，convention应该是介于library和framework之间的某种形态。既不像library一样松散，只负责把所有API扔出来，你想怎么玩都是你的自由；也不想framework一样要求严格，什么东西写在哪里都有明确的规范。BTW：一直以来，我都认为“菜鸟玩framework，高手用library做framework给菜鸟玩”。

其实挺难描述convention到底是个什么东西的，大概就是“我们推荐你这样做，但你如果不这样做，也没啥问题”。废话么？显然不是。对比framework，差异点就在“如果不这样做”上，convention告诉你“没问题”，framework会告诉你“不行”。而在实际的应用中，或多或少都会遇到这样的情况，需要跳出framework的限定去实现需求，这个时候就是“where hack/trick happens”的时间了。

拿asp.net mvc举例，在action返回JSON时，要么每个action都写`return Json(data, AllowGet)`（framework不合理的地方）,要么就多继承一个ControllerBase，自己多写一个`JsonAllowGet(data)`（trick），要么就没有要么了。不愿意用trick，那你就等下个framework的版本吧。（这个问题在mvc4中已经解决，可支持global的配置）

举得例子不一定恰当，因为这问题不能全赖framework的僵化性，还可以让静态语言和闭源背锅。（PS：别说asp.net mvc是开源的，就算是本来就带了一堆的unit test，你敢改么？重编一个可能很多情况下在GAC中的dll，风险略大）说回Backbone自身，各种tutorial里都会告诉你View里最重要的就是render方法了。如果看过源代码，你会很惊奇的发现View的“基类”里的render只是：`render: function(){}`。就是这么简单，而且，没有任何的行为会invoke这个render。那么为什么我不能在每一个View中都实现一个`paint()`，然后以paint作为绘制View的重心完全取代render？当然可以！

罗哩罗嗦写了半天好像看不到任何Convention的厉害之处，因为Convention只是一个前提，下面的一个**H3**下面的内容才是重点。

### Simplicity

Backbonejs一共四个“基类”——Model Collection View Router，其实还有一个关键的Event，已经被mixin到四大基类中了。

Model：包装后的pojo（？），借助Event有了sub/pub的能力，仅此而已		
Collection：model的集合，相比model[]只是多了一些helper而已			
View：所谓核心，源代码行数非常之短，除了delegateEvents以外没啥有用的东西了			
Router：比sammyjs简单很多的一套基于hash的路由		

仔细看过源代码以后，你可以发现Backbone提供的功能简直可以用“简陋”来形容，总代码行数1606行，对比knockout（已经算是简单的MV*了）的4000+行，显得没啥诚意？

“No magic at all”，我觉的这样形容Backbone最为贴切。没有`ko.applyBindings`以后坐等奇迹发生，也就意味着不会有过多的魔法/默认行为。two-way-binding看起来和用起来都很炫酷/简单，但是当app变得庞大时，一切还都像[TodoMVC](http://todomvc.com)一样美好么？

在coding界中，有种东西无色无味却最为致命，曰：坑（pitfall）。一切现在看起来还好的东西，说不定某天都会变成坑。那些framework帮你做了的事情，变成坑的几率，更高。就比如angular的object不用像knockout一样用`ko.observable`封装，写起代码少了很多`property()`，表面上看起来是优雅的阳关道，但却带来了dirty check所引发的performance issue的坑。再比如我用knockout遇到的坑，knockout更像library，组织的太松散了，到底是用多个View还是一个View带很多Subview？持久化和mapping到底怎么做才合适？你需要太多BestPractice来指导才能玩转knockout，但遗憾的是，我目前还没找到（且我自己的practice并不能让我自己满意）。

backbone带来的仅仅是最基础的一些helper和**组织代码和逻辑的建议**罢了，但是这正是恰到好处的。

## Cons
---------------------------------------------------------------
### get & set

因为是Java的无脑低端黑，所以就莫名的讨厌这一对函数，虽然也找不到啥更好的命名了，但在我看来就是cons。

### extend & new

不喜欢在javascript中写出其他静态OO语言中那种`Child : Parent`的感脚，因为跟常见的“按规范继承”不同，javascript的继承是“按实现继承”的（有关理论见《松本行弘的程序世界》）。出现extend和new就觉得别扭得很，况且`new`这种高手们“不齿”的关键字也是非常扎眼。

再加上推荐的file structure也跟传统的静态OO很类似，很可能跟Jeremy Ashkenas本人的背景也有一定的关系（没google到，我猜的）。

不过以后es6成熟和普及了，很多东西可能看起来就没那么怪了，到时候Backbonejs也许还会更火也说不定。

### dependencies

其实我是想帮Backbone洗地来着。相对于其他流行的MV*，一个hard dependency（underscore || lodash）一个recommend dependency（jquery || zepto）似乎有点重了。但在我看来，这两个最流行的库，其实都可以看作是Javascript以后规范里某些必备API的polyfill而已，指不定这些API在es6789就原生支持了，从长远来看，应该不是什么坏事。

不过从现阶段来看，那个叫JQuery的东西，真是越来越大了，着实应该好好黑一下。

### THE future

web的未来应该是属于web component或者是类似的东西的，从这个角度来看，angularjs似乎更能契合web的发展方向。但从另一个角度看，ng-blah或者是data-bind在**现行**的规范中确实还挺烦人了，logic很容易就带入了html里，破坏掉了html/css/js的分离。有种php/asp的既视感，不爽。

## 总结

Backbonejs挺不错的。