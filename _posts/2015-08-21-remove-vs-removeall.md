---
layout: post
title: Remove Vs RemoveAll
date: 2015-08-21
categories: tech
publish: true
---

有个feature需要比较高频率的从集合中移除一部分元素，稍微脑洞了一下，感觉用`for-loop`+`Remove`应该比`RemoveAll`要快（可能是`RemoveAll`的example里用了lambda，影响了判断）。

BUT，test dont lie，写了个测试试了下：

{% highlight csharp %}
var list = new List<int>();
for (int i = 0; i < 1000; i++)
{
    list.Add(i % 3);
}

var iter = 1000;

Profile("Remove", iter, () => 
{
    var l = list.ToList();

    for (int i = l.Count - 1; i > -1; i--)
    {
        if (l[i] == 0)
        {
            l.RemoveAt(i);
        }
    }
});

Profile("Remove All", iter, () => 
{
    var l = list.ToList();

    l.RemoveAll(item => item == 0);
});
{% endhighlight %}

结果`RemoveAll`效率要比`Remove`高好几倍，找了[源代码](http://referencesource.microsoft.com/#mscorlib/system/collections/generic/list.cs,842)扫了一下：`RemoveAll`是`O(n)`且只有一次`Array.Clear`，`RemoveAt`却每次都要`Array.Copy`（压缩空隙的操作），结果显而易见。<del style="color: gray">严格来说，我是在Unity做的测试，但读的是.Net的源代码，好像有点不严谨</del>

**So，Once More：Test dont lie，Code dont lie。**