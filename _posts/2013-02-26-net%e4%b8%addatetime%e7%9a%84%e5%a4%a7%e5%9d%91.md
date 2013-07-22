---
layout: post
title: .Net中DateTime的大坑
categories: tech
tags: []
status: publish
type: post
published: true
---
Long story in short：当使用`JavaScriptSerializer`去序列化一个`DateTime`值的时候，如果这个值的`Kind`是`unspecified`，则会根据`Timezone`去序列化`DateTime`。

这就包含了一个隐含的BUG：如果`DateTime`做了持久化操作在还原时，比如从String “2013-3-1 12:00:00”中初始化`DateTime`的时候，不会有Kind。此时如果在用`JavaScriptSerializer`去序列化，则有可能得到的result是“2013-3-1 12:00:00” +- N

顺手吐槽：有源代码就是好，开始就误解Asp.Net MVC3了，原以为`Json Response`有啥特别的处理，结果发现只是一句简单的

{% highlight csharp%}
if (Data != null) {
    JavaScriptSerializer serializer = new JavaScriptSerializer();
    response.Write(serializer.Serialize(Data));
}
{% endhighlight %}
