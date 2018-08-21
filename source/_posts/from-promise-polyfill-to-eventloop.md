---
title: 从Promise到Event Loop（一）
date: 2018-04-12 15:31:12
description: "如果你要给Promise打补丁的话，你会怎么实现？core-js又是怎么给promise打补丁的？setTimeoute和promise的执行顺序有何差异？"
tags: Javascript
---

# 一切的开始
去年某天，t0在群里扔了一道类似下面的题目（不是这道，这道来自[jake](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/ "jake")）：
```javascript
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});

console.log('script end');
```

当时大家都说出一个理所应当的答案
```javascript
script start
script end
promise1
promise2
setTimeout
```
然而，有细心的同学把用例放到ie8下面跑的时候就会发现结果有些细小的差异：
```javascript
script start
script end
setTimeout
promise1
promise2
```
这里冒出了2个问题：
1. ie8并不支持promise
2. ie8promise的callback执行比setTimeout慢

第一个问题很快被解决，因为打开的页面使用mn（[工程化，开发构建利器]）打包，项目自带了promise补丁 😀   
第二个问题很令人困惑，当时对event loop和microTask的概念还比较模糊，并不能直接找出问题，于是开始怀疑是[core-js](https://github.com/zloirock/core-js)(mn打的promise-polyfill)的promise实现有问题，和原生的表现不一致，从而开始拜读corejs的实现代码（俄罗斯大哥真的牛逼，一个人维护整个core-js）

# Promise Polyfill
> promise的polyfill有多个实现包，包括著名的bluebird, 但是这里只讨论core-js的实现

照例先上一段代码：
```javascript
// core-js/modules/_microtask.js

  // Node.js
  if (isNode) {
    notify = function () {
      process.nextTick(flush);
    };
  // browsers with MutationObserver
  } else if (Observer) {
    var toggle = true;
    var node = document.createTextNode('');
    new Observer(flush).observe(node, { characterData: true }); // eslint-disable-line no-new
    notify = function () {
      node.data = toggle = !toggle;
    };
  // environments with maybe non-completely correct, but existent Promise
  } else if (Promise && Promise.resolve) {
    var promise = Promise.resolve();
    notify = function () {
      promise.then(flush);
    };
  // for other environments - macrotask based on:
  // - setImmediate
  // - MessageChannel
  // - window.postMessag
  // - onreadystatechange
  // - setTimeout
  } else {
    notify = function () {
      // strange IE + webpack dev server bug - use .call(global)
      macrotask.call(global, flush);
    };
  }
```
代码很简单，通过环境，浏览器的版本等因素找到一个最适合的promise的实现，具体思路如下：
### 1. node4+ 直接使用process.nextTick
nextTick是node的私有方法，允许在当前调用栈结束立刻执行回调。
### 2. 在支持MutationObserver的浏览器使用Observer
MutationObserver用于监听DOM节点，当节点发生变化的时候，执行回调函数。(MutationObserver同样在vue源码中用上了~)当前MutationObserver的proposal已经被废弃。
![MutationObserver](/images/eventloop/WX20180327-210644.png)
### 3. 当前环境已经有打promise的补丁的话，直接使用promise补丁
因为如果不支持1，2两种方式的话，说明完全标准的promise补丁已经没法实现了，这个时候只能委屈求全了。
### 4. 当以上情况都不适用，使用下面的几种backup
 - node0.8-的process.nextTick([node0.8-的process.nextTick属于macroTask](https://github.com/nodejs/node/wiki/API-changes-between-v0.8-and-v0.10))
 - Sphere(游戏引擎) 的Dispatch方法
 - MessageChannel（多线程通信, 双向，支持webworker）
 - window.postMessage (单向, 支持webworker)
 - IE8-监听节点的ONREADYSTATECHANGE事件
 - setTimeout (最后的无奈)

补一张图片说明：
![promise-polyfill](/images/eventloop/Promise polyfill.png)

至此可以清晰的了解到core-js的补丁思路，当1，2行的通的时候我们会得到一个“相对标准”的promise。3，4能让我们得到一个“勉强实现的promise”
这里回到我们的问题，当文章的示例脚本跑在IE8的时候，很明显我们逻辑会走到ONREADYSTATECHANGE这儿，那么为什么监听的事件回调为什么比setTimeout慢执行呢？是不是所有的事件回调都比setTimeout慢呢？http请求的呢？到底异步队列是怎么实现的？标准如何？

故事很长，下回再续。

<!-- # 下回
[从Promise到Event Loop（二）](http://km.weoa.com/group/fe/article/3474) -->

# 参考资料
[MutationObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/MutationObserver)
[Caniuse](https://caniuse.com/#search=MutationObserver)
[macroTask and microTask](https://github.com/YuzuJS/setImmediate#macrotasks-and-microtasks.)







