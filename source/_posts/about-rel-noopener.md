---
title: "[译] 关于rel=noopener"
date: 2018-08-21 12:14:37
description: "有关rel=noopener的一些情报"
tags: HTML
---

> Original(原文地址)：https://mathiasbynens.github.io/rel-noopener/

## 他解决了什么问题？
假设你现在正在浏览`index.html`，有一个由用户生成的链接在你的网站里面：

> <a href="/test-rel-noopener/" target="_blank">Click Me!(同域)</a>

点击上面的链接会打开一个新页面（使用`target=_blank`）。就这个参数本身而言，并没有值得兴奋的东西。

然后新打开的页面有一个`window.opener`对象指向你现在正在浏览的页面`index.html`。这就意味着一旦用户点击了链接，另外一个页面就能完完整整的控制本窗口的window对象了。

即便是越狱的情况下`window.opener.location`一样是可以访问的，CORS并没有控制到这里（注： 从最新的chrome测试情况看，跨域情况下`opener.location`不能获取数据，可以replace操作，跳转的页面还是可以强迫主页面跳转到一个指定的页面）

跨域的例子：

> <a href="https://genuifx.com/github/blog/about-rel-noopener/" target="_blank">Click Me!（跨域）</a>

作为证明，一旦打开了新页面，本页面会多一个`#hax`的hash后缀。这只是一个无害的列子，如果它重定向到钓鱼页面，并且要求用户输入登录信息的话，用户很大可能不会注意到页面已经变了，因为页面的跳转的时候用户停留在打开的窗口上，此时旧页面的跳转是在后台进行的。如果在重定向到钓鱼页面之前添加一点延迟的话，这种攻击就显得更加微妙了（[标签抓取](http://www.azarask.in/blog/post/a-new-type-of-phishing-attack/)）

（注：标签抓取 tab nabbing 原文指的是用户一旦浏览其他标签页，钓鱼网页在本地里就替换自己的ico和网站标题，例如Gmail的图标+您收到一份邮件的标题。并且把页面改造成类似用户登录的页面， 这种情况下由于页面都是后台更新，用户很容易就上当受骗。）

## 推荐做法
为了防止页面滥用`window.opener`对象，Chrome49和Opera36中，可以使用`rel=noopener`来确保`opener`为空。

> <a href="/test-rel-noopener/" target="_blank" rel="noopener">Click Me!(noopener)</a>

一些旧的浏览器可以使用`rel=noreferrer`（同时会禁用HTTP的Referrer头）。或者使用js禁用：
```
var otherWindow = window.open();
otherWindow.opener = null;
otherWindow.location = url;
```

> <a href="/test-rel-noopener/" target="_blank" rel="noreferrer">Click Me!(noreferrer)</a>

> <a href="/test-rel-noopener/" target="_blank" onclick="var otherWindow = window.open();otherWindow.opener = null;otherWindow.location = href;">Click Me!(jsopen)</a>

需要注意的是js的方式在Safari不可用。需要支持Safari的话，可以插入一个iframe来打开新的页面，然后马上删掉iframe。

除非[有理由](https://css-tricks.com/use-target_blank/)，尽量少使用`target=_blank`，特别是在用户生成的内容地方。

> [caniuse](https://caniuse.com/#feat=rel-noopener)


> Original(原文地址)：https://mathiasbynens.github.io/rel-noopener/