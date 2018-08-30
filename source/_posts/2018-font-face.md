---
title: "2018年的@font-face"
date: 2018-08-28 20:34:39
description: "字体攻略、font-face详解，以及2018年的前端应该怎么使用font-face"
tags: [CSS, FONT, Font-Face]
---

说起自定义字体`font-face`，讲真的其实不大熟悉，用的最多的时候是定义iconfont，语法也基本不需要自己写，要么用postcss插件，要么在生成iconfont的时候自动生成对应的字体定义。除掉iconfont，对字体方面的东西实在是知之甚少，毕竟在经历的多个公司项目中，大多数项目偏向移动端，Hybrid。直到某天，同事给我发了个截图，询问我关于字体压缩问题，我看着截图里面的PingFang字体(10MB)的时候目瞪狗呆，不禁感慨：时代在进步啊，现在的PC端都能这么玩了，当年我们全站800KB都觉得对不起用户。 。

那么在生产项目中到底能不能用上自定义字体？我们的用户能接受这样的流量消耗吗？

> [这是一个测试连接，流量够的点 X)](https://genuifx.com/github/font-test/index.html)

但就PC应用而言，加载一个5MB的字体只需要几百毫秒（网线直连 chrome：600ms+；小米wifi chrome：5s+），用户能够明显的感受到网页由于字体加载变慢了。移动端的话，随着流量越来越廉价，为了更优化的体验，这点流量似乎也能接受。（毕竟也只是第一次加载才需要完整下载，其他时候一般是304）

另外中英文字体大小差距也相当明显，中文字符（6w+）远大于英文字符集，[google-font](https://fonts.google.com)上面的英文字体大多数在1M到2M之间（woff2），~~而中文字体譬如PingFang体积到5.3M（woff2）~~，一个7718字符的中文字体包1.1MB（woff2）, 完整的中文字符集怎么也得10MB了。我尝试在网上找到一些免费的中文字体有大有小，另一个角度看的话，7000+个常用中文字体对于一般的网站来说又似乎足够用了。

既然自定义字体未来在生产项目中可以期待，那么必要的了解是必须的。

## 字体分类
字体的种类相当的多，有OpenType、TrueType、Type_1等等。一般web用到的有otf、ttf、eot、woff(2)、svg。

### TrueType（.ttf）和OpenType（.otf）
TrueType字体是Type 1(Adobe公司开发)的竞品，由苹果公司和微软一起开发，是mac系统和window系统用的最广泛的字体，一般从网上下载的字体文件都是ttf格式，点击就能安装到系统上。

TrueType是OpenType的前身，90年代微软寻求苹果的GX排版技术授权失败，被逼自创武功取名为TrueType Open，后来随着计算机的发展，排版技术上需要更加具有表现力的字体，Adobe和微软合作开发了一款新的字体糅合了Type 1和TrueType Open的底层技术，取名为OpenType。后来OpenType被ISO组织接受为标准，称之为Open Font Format（off）。

ttf和otf字体在web端来说兼容相对较好，除开IE（很神奇不）和早期的ios safari和Android不怎么支持外，其他浏览器都兼容都不错。然而虽然它们俩在桌面端混的很好，到了移动端却不怎么吃香，因为他们的字体文件实在是太大了。。PingFang-SC-Regular字体ttf格式11.3MB而otf要到21.4MB

![](/images/font-face/caniuse-otf-ttf.png)

### Embedded OpenType(.eot)
刚刚提到otf字体虽然在桌面混的风生水起，但是由于体积太大，在web端实在是吃不香，于是微软就是根据OpenType做了压缩，重新取名为Embedded OpenType，顾名思义，这个字体就是为了嵌套到网页做的。后来微软多次给W3C申请，想把EOT当做官方推荐字体，但是最后都没有成功，反倒是WOFF后来居上，成为标准字体。

EOT是微软的亲儿子，古董IE浏览器都支持这个格式，但是其他主流浏览器并不支持EOT。另外就是eot字体也比ttf文件要大一些，所以除非是为了IE8-的浏览器兼容，完全没有必要用这东西。

![](/images/font-face/caniuse-eot.png)

### Web Open Font Format（.woff）
woff在09年开始开发，12年就成为w3c的推荐标准，由Mozilla基金会，Opera Software以及微软主导。

在woff基础上，woff2优化了压缩算法，在[字节码级别](https://en.wikipedia.org/wiki/Brotli)做了压缩。woff2在2018年3月就成为了w3c的推荐格式。

woff字体最大的优点就是体积小，同样的字体对比EOT要小20%到40%，woff2更是比eot、ttf小了50%。由于从12年开始就成为了W3C的推荐标准，woff在主流浏览器的支持程度可以说非常的好了！woff2因为刚成为标准，兼容性稍差，但是主流浏览器都有计划支持它。

![](/images/font-face/caniuse-woff.png)

## font-face语法
哔哔了这么多，总算对web字体有了一个大致的了解。再来看看font-face的语法：
> `@font-face { font-family: <identifier>; src: <fontsrc> [, <fontsrc>]*; <font>; }`
> `<fontsrc> = <url> [format(<string>)]`
> 引用自[css小册](http://css.doyoe.com/)

有几个关键的标识符：
- `url()`: 资源地址，可以加或者不加引号
- `local()`: 本地资源标识符，譬如`local('simsun')`指向本地的宋体
- `format()`: 指定字体格式

所以一个自定义字体定义一般是这样的：
```css
@font-face {
    font-family: "ChangChengChangSongTi";
    src: url("./font/ChangChengChangSongTi/ChangChengChangSongTi-1.woff2") format("woff2"),
        url("./font/ChangChengChangSongTi/ChangChengChangSongTi-1.woff") format("woff");
}
```

然而，事情并不那么简单。

## font-face的兼容写法
实际开发中，我们总是能看到如下的写法：
> code from [postcss-font-magican](https://github.com/jonathantneal/postcss-font-magician)
```css
@font-face {
   font-family: "Alice";
   font-style: normal;
   font-weight: 400;
   src: local("Alice"), local("Alice-Regular"),
        url("http://fonts.gstatic.com/s/alice/v7/sZyKh5NKrCk1xkCk_F1S8A.eot?#") format("eot"),
        url("http://fonts.gstatic.com/s/alice/v7/l5RFQT5MQiajQkFxjDLySg.woff2") format("woff2"),
        url("http://fonts.gstatic.com/s/alice/v7/_H4kMcdhHr0B8RDaQcqpTA.woff")  format("woff"),
        url("http://fonts.gstatic.com/s/alice/v7/acf9XsUhgp1k2j79ATk2cw.ttf")   format("truetype")
}
```

看到一堆代码是不是有点头大？但其实这个定义也很简单，首先看本机是否已存在字体，没有的话依次加载指定格式的字体文件。但是其中蕴含了很多前辈的心血和努力。

### IE8-的兼容
由于IE8-的浏览器只支持EOT格式的字体， 那么我们似乎只要这么定义就完事了：
```css
@font-face {
    font-family: "My font";
    src: url("./path/to/font.eot");
}
@font-face {
    font-family: "My font";
    src: url('./path/to/font.otf');
}
```

神奇的是，IE虽然不支持otf格式但还是会下载文件下来，这可是10MB+啊，这么写是肯定不行了。

[在IE8-打开会有神奇的事情发生](https://genuifx.com/github/font-test/ie8.html)

于是Paul Irish在总结了两种跨浏览器的兼容方案，被各种插件，开发者广泛的应用。

### Bulletproof font-face
第一种方案是利用`local()`，由于IE不认识local标识，IE会自动忽略后面的字体定义，进而不会引发多余字体的下载。

> [Smiley variation](https://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/)
```css
@font-face {
font-family: 'Graublau Web';
src: url('GraublauWeb.eot');
src: local('☺︎'),
        url('GraublauWeb.otf') format('opentype');
}
```

这里用一个指定了笑脸，IE只会解析到eot文件，而其他浏览器则能够正确的解析到otf文件。

另外一种方案不需要`local()`，而是利用`?`符号帮助IE做字体解析：

```css
@font-face {
  font-family: 'Graublau Web';
  src: url('GraublauWeb.eot?') format('eot'), url('GraublauWeb.woff') format('woff'), url('GraublauWeb.ttf') format('truetype');
}
```

## 2018年的font-face写法
Paul最后一次更新在2010年，文章到现在已经7、8年了。浏览器经过无数迭代，woff2成为了新的W3C推荐标准，大量的浏览器兼容了woff，实际上我们已经不需要每次定义字体都写上密密麻麻的代码。简单的两句话就可以支持大部分的浏览器了
```css
@font-face {
    font-family: "My font";
    src: url("./path/to/font.woff2") format("woff2"),
        url("./path/to/font.woff") format("woff");
}
```
这种写法在IE9+、IOS5+、Android4.4+以及所有现代浏览器都能很好的运行。

如果要支持Android4.4以下的设备，就得加上otf文件支持，但正如上述，otf文件太大，在低端安卓机子表现估计也不好，还不如对这部分用户做优雅降级。

最后推荐一个字体格式转化的网站[font-converter](https://font-converter.net/)，免费而且可以一次生成多种格式（包括woff2）的文件，⚠️但注意千万不要直接使用该网站生成font-face规则。

⚠️值得注意的是，不少字体不是给web专门设计使用，使用它们有可能面临体积过大或者版本问题，可以使用[wakamaifondue.com](https://wakamaifondue.com)来检查字体文件的情况。

## 参考资料
[Embedded_OpenType](https://en.wikipedia.org/wiki/Embedded_OpenType)
[OpenType](https://en.wikipedia.org/wiki/OpenType)
[Web_Open_Font_Format](https://en.wikipedia.org/wiki/Web_Open_Font_Format)
[TrueType](https://en.wikipedia.org/wiki/TrueType)
[Bulletproof @font-face Syntax](https://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/)
[Using @font-face - CSS-Tricks](https://css-tricks.com/snippets/css/using-font-face/)