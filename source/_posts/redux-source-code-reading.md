---
title: redux 源码阅读
date: 2019-10-08 18:46:17
description: "redux 源码阅读, 实现细节分析"
tags: Javascript
---

## 开发和打包
使用 `typescript` 编写源码，使用rollup打包输出。

`ts` 相关配置启用了严格的类型检查，自动输出声明文件到 `types` 目录，自动输出sourcemap文件，指定了目标编译版本是 `esnext`，使用的模块系统是 `ESM`，此处输出选择了 `ESM`，应该是为了后续 rollup 打包的时候能够做 TreeSharking. 

`rollup` 配置的是发布的内容，redux 生成了 CommonJS 格式的代码到 `lib` 文件夹下（默认 node 检索依赖会查找 `lib` 目录）, 生成了 UMD 格式的代码到 `dist` 目录，另外还为现代浏览器生成了 `ESM` 格式的代码到 `es` 目录下。值得一提的一个细节是，rollup 配置了输出多个目标非常简洁，另外 redux 还特别注意了一次打包只生成一个声明文件。

既然生成了多个目标版本的 redux 源码， 那么使用的时候该用哪个版本呢？这里 redux 在 package.json 中指定了：

![9d79039cb99df2dce0e2fa31ecef9323.png](/images/redux-source-code-reading/1.png)

其中 UMD 的版本是为了发布到 [`unpkg`](https://unpkg.com) 平台去。

## `createStore`
创建一个 `store`，包装函数后返回。

朴实无华的实现，利用闭包的特性，使得 store 只能通过局部作用域的指定函数去修改，也就是 dispatch。

另外一点有趣的是，如果有enhancer（也就是 中间件 ）存在的时候，此时创建 store 的任务就交给了 enhancer。
![a6943de5d7bf11ffc9c312a38b70602a.png](/images/redux-source-code-reading/2.png)


## `combineReducers`
转化一个 object 成为一个 reducer 函数，因为 createStore 只能接收一个 reducer 函数，当需要将 reducer 切分为模块去使用的时候就需要 combineReducers 去处理其中的逻辑。

有两点细节处理：
1. 对于非法的 reducer 返回值，使用 cache 保存其报错，保证每次更新的时候不会重新的报错。
2. 如果每个reducer的返回都没有变（shallow equal）那么直接返回上一个 state。

## `compose`
从右到左依次调用函数，上一个函数的输出作为下一个函数的输入，类似pipe。 `(...args) => f(g(h(...args)))`

## `applyMiddleware`
创建一个 store 的 enhancer。

又是一个朴实无华的实现，创建 store，然后调用 compose 依次执行中间件的逻辑，每个中间件能拿到当前 store 的 state，并且可以包装dispatch函数。

### `redux-thunk`
如果 action 是一个函数，表示此时把 dispatch 的控制权下放给action，否则直接执行 dispatch。

### `redux-logger`
在每次 dispatch 之后主动的打印 log。

## `bindActionCreators`
用的比较少，也比较简单，将多个 action 合并成为一个大的 object 去使用。