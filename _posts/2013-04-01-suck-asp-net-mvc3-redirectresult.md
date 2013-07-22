---
layout: post
title: 坑爹的ASP.Net MVC3 RedirectResult
categories: tech
date: 2013-04-1
---

他代码是这样写的：

{% highlight csharp %}
[SuppressMessage("Microsoft.Design", "CA1054:UriParametersShouldNotBeStrings", MessageId = "0#", 
Justification = "Response.Redirect() takes its URI as a string parameter.")]
public RedirectResult(string url, bool permanent) {
    if (String.IsNullOrEmpty(url)) {
        throw new ArgumentException(MvcResources.Common_NullOrEmpty, "url");
    }

    Permanent = permanent;
    Url = url;
}
{% endhighlight %}

然后metadata里的注释写的是：

{% highlight csharp %}
// Summary:
//     Initializes a new instance of the System.Web.Mvc.RedirectResult class.
//
// Parameters:
//   url:
//     The target URL.
//
// Exceptions:
//   System.ArgumentNullException:
//     The url parameter is null.
public RedirectResult(string url);
{% endhighlight %}

我就呵呵了，不是ArgumentNullException么，为啥连`string.Empty`也抛？