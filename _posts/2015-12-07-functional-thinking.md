---
layout: post
title: Functional Thinking
date: 2015-12-07
categories: tech
publish: true
---

标题起得有点大。

说不上来到底是幸运还是不幸，总之就是年底有空超前完成了一下明年的KPI——Learning Haskell。完成度还算满意，就连传说中的**monad**也算是有个感性的认识了。

强推[这个教程](http://learnyouahaskell.com/)，基本上是我所有看过的教程里最良心的了。

第一次听说**Functional Programming**还是好几年前刚刚开始写代码营生的时候，那时的我还是初级代码搬运工，冷不丁搬运到了`.Where(=>).Select(=>).ToArray()`这种第一眼看不明白多看两眼又觉得好厉害的代码，瞬间就惊为天人。这好几个`for`都写不出来的代码，一个chain就做完了。顺手多google了一下linq的东西，也就无可避免的学到了一个新单词——**Functional Programming**。

平凡人类认知那些first impression不错新事物，大多会经历从overestimate到underestimate的一个或多个反复。反正我对C#中的linq的认知是经历了“linq == silver bullet”和“linq == performance hit”的起伏的，现如今再看，当然是能明白具体问题具体分析才是重点，只是写过的那些代码<del>坑</del>里就不免一时都是linq又一时连`using System.Linq`都不见。

不过对于FP的好感一直是有，一是因为不懂，二是因为直观看来代码确实精简。这两年给自己安排的KPI都是FP相关的（去年是lisp，今年的lua相关性弱一点，之后的haskell可是自喻为**pure FP**呢），嗯，这叫IDL（Interest Driven Learning）。

今年眼看就要结束，总算是能沉淀一些在我看来能算作是Thinking的东西，还挺好的。

<!--more-->

### Blah Blah Blah

有关Haskell的**Purity**——多么不容易出BUG，多么适合Concurrency；**Laziness**——多么高效，你随意google两篇文章都会吹的天花乱坠，毕竟现在语言这么多，要想出头都要有能抓眼球的卖点。

只是凡事都有两面性，purity带来的问题——把很多在imperative式的语言中极其简单的操作——比如`random()`——会变得复杂到要用**monad**来解决了；laziness带来的问题——在一些情况下可能变成了performance hit，解决办法也挺复杂。

所有slogan和poster都不能尽信，不限于programming language。

Blah Blah Blah

### Devil is in the details

从空格开始说起。

都说用Ruby来做DSL（Domain Specified Language）简直天造地设，一大原因是ruby的函数调用不用写括号（`func(1)`可以写作`func 1`），所以有些库用起来就感觉是新定义了一些语法似的（实际只是do block加函数调用而已）。

在Haskell中，函数调用只需要一个` `，`func(1)`已经是错误的语法了，因为括号已经回归到了它数学含义——优先级（precedence）（当然还有`IO ()`）

抄段代码，可以一窥只用空格（或者说是更加函数式）的代码有多简洁（嗯，确实是6行版快速排序，不要管那些“奇奇怪怪”的语法，感性认识就好）：

{% highlight haskell %}
quicksort :: (Ord a) => [a] -> [a]    
quicksort [] = []    
quicksort (x:xs) =     
    let smallerSorted = quicksort $ filter (<=x) xs
        biggerSorted = quicksort $ filter (>x) xs
    in  smallerSorted ++ [x] ++ biggerSorted  
{% endhighlight %}

中间的`smallerSorted`这个helper function用带括号的版本大概需要这样写（in js）：

{% highlight javascript %}
function smalllerSorted(x, xs)
{
	return quicksort(filter(function(y){ return y <= x; },xs));
}
{% endhighlight %}

FP本来就是偏学术的语言，理所应当简洁而严谨（OpenOffice真难用）：

![ivf](/assets/images/ivf.png)

FP从Lisp开始一脉相承，从来就是写的少做的多（Write Less，Do More，jQuery笑了），[wiki](https://en.wikipedia.org/wiki/Declarative_programming)上直接将FP划做是Declarative programming的一个子类，也是有点道理的。

Lisp的代码本来就**简洁**，只是去掉了Lisp中那被人诟病的许许许许多多多多的括号了以后（当然除了` `，还有`$`和`.`的功劳），Haskell的代码自然就变得更**漂亮**了。

找不到那张黑的更厉害的图了，这张是一个意思：

![excerpt](/assets/images/lisp-vs-javascript.png)

### Apply over call

有函数fn:

{% highlight haskell %}
fn x y z = x + y + z
{% endhighlight %}

那么

{% highlight haskell %}
fn 1 2
{% endhighlight %}

是个啥东西？当然在多数语言中，已经报错了。不过“数学”一点的想，带入`x=1 y=2`，`fn`的右边就变成了`1 + 2 + z`。所以`fn 1 2`就成了

{% highlight haskell %}
fn2 z = 3 + z
{% endhighlight %}

这就是为什么我觉得FP的函数调用（Call）都用Apply这个单词更为合理（在看lisp的时候也是这种感觉），因为函数的调用并不是得到一个值就完了，而是返回了一个新的**东西**（值，函数，或者context）。这种高级的特性有个学名——柯里化（Currying），顺带一提，javascript也能使用这个特性，只不过又丑又麻烦就是了（我不是针对谁，只是针对es6以前的javascript）。

再“数学”点想，`fn2`的等式两边都有一个`z`，能不能直接都划去了？还真能：

{% highlight haskell %}
fn2 = (+) 3
{% endhighlight %}

`(+) 3`只是把中序表达式转回了前序，不难理解。我们从三个参数推到了零个（？），是不是特别的reasonable？综上可证：*所有接受n个参数的函数，都可以看作先接受1个参数生成的新函数，他接受n-1个参数*

Math over programming？对，就是这个feel。

这才是真正的高阶函数（high-order），不只是可以把函数当参数传递这么简单。

### Patern matching

`if else`挺讨厌的，嵌套起来更甚。只要读过代码（别人的，或者自己昨天以前写的），都会懂在一个函数里看到许许多多的branch是多么令人绝望，因为他往往伴随着bug、潜在的bug、大洪水和诸神黄昏。在工程上衡量代码复杂度的一大指标[Cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)简单理解就是branch，看，它讨厌（我们都讨厌复杂的东西）还是有科学依据的。

模式匹配可以帮你消灭一部分的`if`：

{% highlight haskell %}
--fibonacci seq
fib 0 = 0
fib 1 = 1
fib 2 = 1
fib n = (fib $ n - 1) + (fib $ n - 2)
{% endhighlight %}

如果抛弃`=`就是赋值（assign）的“传统观念”，上面的代码段还是挺好理解的——输入0、1、2的时候返回固定值，其他输入递归返回求和结果。原来在《七周七语言》中看过了prolog解决地图着色问题的例子，所以再看模式匹配的代码，神奇的感觉少了不少，反而更能察觉到模式匹配的实用性了：

{% highlight haskell %}
--should be Maybe
--return 1st elem of a list
myFirst :: [a] -> a 
myFirst (x:_) = x
{% endhighlight %}

只关心list的第一项的时候，我真想不到还有写法比`(x:_)`更加简单直白。而且，用“天生通配符脸”的`_`来匹配那些不关心的东西，也是很搭。

还有个细节：模式匹配匹配不到的时候，会报错，比如`myFirst []`（因为`x:`只会匹配到非空list）。对应到imperative中，就是最后那个`else`中的`throw new UnreachableException()`，虽然很多时候我们并不会care到这个最后的分支，从而带来一些bug。

说回`x:_`，为啥`x`只匹配到了一个元素，而不是更多？因为其实`:`只是一个语法糖，实际上就是haskell中`cons`。

终于写到我最爱的部分了。

### Sugar Sugar Sugar

这些年的越来越深有体会——**语法糖是一个语言是否能流行/是否能用的顺手的关键点**。

其实，Haskell中的很多“语法糖”并不是标准的“语法糖”，并不是给编译器看的，而就是一些函数的定义，只是用上了各种符号而已。

先看一段代码：

{% highlight haskell %}
[1,3,5] >>= \x -> [2,4,6] >>= \y -> return (x,y)
{% endhighlight %}

是不是感觉画风一变，跟上面那些看一眼就能猜个大概的代码段完全不一样了，完全猜不到输出。而且如果`>>=`就是某种语法糖的话，这只怕是苦瓜糖吧。

嗯，`>>=`它，确实不是语法糖。他只是个神奇的函数：

{% highlight haskell %}
class Applicative m => Monad (m :: * -> *) where
  (>>=) :: m a -> (a -> m b) -> m b
{% endhighlight %}

更加看不懂了对不对？不过你应该看到了一个关键字——`Monad`，传说中的**Monad**！不过这篇文并不想多讲Monad（我也讲不出什么东西），你只要知道上面那个例子中的`>>=`就是Monad的list实现版本就好。

让我们忘记Monad，先把那个例子等价地再写一遍：

{% highlight haskell %}
do x <- [1,3,5]
   y <- [2,4,6]
   return (x,y)
{% endhighlight %}

好像清晰了不少，也大概能一猜输出结果了。看来`do`和`<-`有点作用。

接下来是另一个等价的版本：

{% highlight haskell %}
[ (x,y) | x <- [1,3,5], y <- [2,4,6] ]
{% endhighlight %}

我想你差不多能猜到输出了：

{% highlight haskell %}
[(1,2),(1,4),(1,6),(3,2),(3,4),(3,6),(5,2),(5,4),(5,6)]
{% endhighlight %}

这就是Haskell中的list comprehension，其实他只是list monad的语法糖而已。但我觉得应该没有人写haskell会喜欢用`>>=`而不用list comprehension。

集合是每个编程语言中都需要特殊处理的绕不开的问题，试想有两篇tutorial，一篇从list comprehension开始，而另一篇直接从list monad讲起，前面还必须得把`data` `class`这些语言元素都讲明白。这两个tutorial哪个更能“拉新”，不言而喻。

忘了在哪儿看到过，某大神说他写js就是喜欢用`''`而不是`""`，原因是少按一次shift键。没人不喜欢更少的keystroke和更清晰的表达。

还有两个前面就提到过的小玩意——`$`和`.`，直接用教程里的例子：

{% highlight haskell %}
oddSquareSum :: Integer  
oddSquareSum = sum (takeWhile (<10000) (filter odd (map (^2) [1..])))
{% endhighlight %}

可以简化成：

{% highlight haskell %}
oddSquareSum :: Integer  
oddSquareSum = sum . takeWhile (<10000) . filter odd . map (^2) $ [1..] 
{% endhighlight %}

直观的感受就是省略了很多括号，说详细点`.`用来做function composition，把`a -> b`和`b -> c`组合成`a -> c`；而`$`就是函数的apply，只是把优先级改变了。括号并不是毒药，只是太多的括号看起来会比较讨厌，当代码不得不讨厌的时候，你就需要这些小玩意了。

当然最后作者最后还给出了一个*However, if there was a chance of someone else reading that code, I would have written it like this*的版本：

{% highlight haskell %}
oddSquareSum :: Integer  
oddSquareSum =   
    let oddSquares = filter odd $ map (^2) [1..]  
        belowLimit = takeWhile (<10000) oddSquares  
    in  sum belowLimit 
{% endhighlight %}

嗯，有时候甜的吃多了会蛀牙，**可读性**还是要放到第一位的。

### Type

Programming is all about Abstraction。

每一种语言或者是范式（programming paradigm）都在实践一些特定的抽象思路，OOP的思路就是针对特定数据和对应的方法，抽象出**类**或者是**对象**。而数据，又可以描述为状态（state），必须是可变的。某些FP的信徒觉得这是有害的，是难以debug的，是root of all bugs，所以FP就只抽象**方法**咯？就我理解：对也不对。说对，是在haskell中的`data`和`class`的定义中，你确实只能定义方法，找不到地方能写数据。说不对，因为`data`本身就是在抽象数据，只是更像数据结构，而不是状态。

一个简单的二叉树的定义的两个版本：

{% highlight csharp %}
//c#
class Node<T>
{
    public T data;
    public Node<T> left;
    public Node<T> right;
}

class Tree<T>
{
    public Node<T> root;
}
{% endhighlight %}

{% highlight haskell %}
--haskell
data Tree a = EmptyTree | Node a (Tree a) (Tree a) deriving (Show, Read, Eq) 
{% endhighlight %}

对比C#版本，haskell的代码是不是很像数据结构教程里的伪代码？

再多看两眼haskell的实现，你会有两个直观的感受——类（类比）定义竟然都能递归；泛型如此的平常。函数式语言，递归稀疏平常，无所惊奇。只是泛型二字，值得多说两句。

首先，应该在所有的haskell文档中都找不到“泛型”，因为这是我自己编的…………不过这个概念对应到主流语言中，也就泛型比较合适。

看个函数的类型定义：

{% highlight haskell %}
func :: a -> a
{% endhighlight %}

读作函数`func`，传入一个参数，返回一个相同类型的结果，至于到底是什么类型，并不做限制。像不像：

{% highlight csharp %}
T Func<T>(T a)
{% endhighlight %}

而且haskell中的`class constraint`也跟C#中的`where`很像，只是没有特定的关键字，只要：

{% highlight haskell %}
toString :: Int -> String
{% endhighlight %}

可以说haskell中类型限制更像弱类型里的*duck typing*。简单就能推断出来的类型，就没有在代码里写明的必要，编译器能做的事就都交给编译器，这才是编程语言正确的发展方向。

看回二叉树的那个例子：

{% highlight haskell %}
data Tree a = EmptyTree | Node a (Tree a) (Tree a) deriving (Show, Read, Eq) 
{% endhighlight %}

还有个特别的关键字`deriving`，如果只从字面理解就是继承，不过后面的东西只能是`typeclass`——如果要强行类比的话——就是`interface`，不过又不太一样：

{% highlight haskell %}
class Eq a where  
    (==) :: a -> a -> Bool  
    (/=) :: a -> a -> Bool  
    x == y = not (x /= y)  
    x /= y = not (x == y)  
{% endhighlight %}

`typeclass`是可以带默认的实现的！嗯，这就不像`interface`而更像ruby中的`module`了。

看，haskell能做到像弱类型一样的灵活，可他偏偏还是个正宗的强类型静态语言。

### FP is coming

随意贴几个图：

Google suggestions：

![js](/assets/images/fp-js.jpg)

![py](/assets/images/fp-py.jpg)

![csharp](/assets/images/fp-csharp.jpg)

Google Trend（keyword == functional programming）：

![trend](/assets/images/fp-trend.jpg)

FP有些天生优势确实挺适合工业级的开发的，所以各种主流语言汲取或多或少的FP元素不难理解，最近几年FP越来越热也是事实。

IMHO：现在FP（Haskell也好，lisp也罢）真正成为主流语言只是少了一个killer级别的framework和应用，罢了。