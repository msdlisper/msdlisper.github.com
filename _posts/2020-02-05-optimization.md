---
layout: post
title:  "web性能优化实践"
date:   2020-02-05 00:30:00
categories: learn
tags: 性能优化
---
* content
{:toc}

性能优化, 从和说起



## 引入

在软件开发行业, 优化是永恒的话题, 特别是前端领域, 性能优化总是和用户体验挂钩. 我们的页面加载的越快, 页面反应越快, 用户体验越好, 当然用户体验包括其他很多点.

我目前主要是在移动端webapp方向积累了一些优化经验

## 了解浏览器的几个时间节点

我们可以通过Navigation Timing API 来获取各个时间节点, 也就是通过window.performance.timing来获取

- 起点: 一般使用fetchStart, navigationStart是包括前一个页面卸载用的时间
- 白屏时间: 一般是domLoading-fetchStart, 头部资源加载完成
- 首屏时间: dom树已经解析完成, 一般是domContentLoadEventEnd-fetchstart, 也就是常说的domready时间. 
- 所以资源加载完成时间: loadEventEnd-fetchStart, load事件的事件

上面指标,对于SPA应用, 其实意义不是很多, 当然现代浏览器已经有针对SPA的一个统计指标, 这些用chrome的performace工具可以得到分析:

- FP/FCP 首次绘制, 这个时候: 页面一般会呈现出loading图标
  - 通过服务端渲染, 内联css可以加快
- DCL/LOAD 首次渲染: 页面一般会出现footer, 或者顶部banner
  - 通过CDN加速, 缓存, 以及GZip来提高
- FMP 首次有效绘制/可交互状态时间TTI: 这个时候出现了页面架子, 但图片啥的可能没加载完
  - 通过精简业务逻辑, 路由拆包来提高
- 加载完成: 图片啥的都加载完成了
  - 图片压缩, webp来提速

<img src="http://ww1.sinaimg.cn/large/ba0e41a3ly1gbmzzelkr3j20ov078n3g.jpg" width="600px"/>

这个可以参数[浏览器术语](https://mp.weixin.qq.com/s/GIpmZIY6yxGRBpkTDHuJuw)



## 优化js的加载速度

如果不做任何优化, js文件的下载和执行都还中断html parser, 如果header里的script文件太大, 白屏的时间就会很长


接下来了解 prefetch, preload, dns-prefetch, defer和async

### prefetch
形如 `<link href="main.js" rel="prefetch">`, 浏览器利用空余时间来加载js, 存储在disk里, 当浏览器某一时刻发现script里用了main.js, 就从disk里面取.

适合对于非首屏用到的js文件, 提前加载好需要用的js资源

使用的谨慎, 对于页面马上要用的资源, 不要用prefetch, 不然js没下载完, 又要用这个js, 浏览器又会再次请求

### preload
形如`<link rel="preload" href="/main.js" as="script">`, 浏览器遇到这个标签就立即加载main.js, 这个过程不阻塞页面parser, 等要用到main.js的时候, 就从内存里读取disk

这个适合首屏的js用, 可以结合webpack的插件:ScriptExtHtmlWebpackPlugin

### dns-prefetch
形如: `<link rel="dns-prefetch" href="//cdn.com/">`. 预先解析cdn的DNS地址, 从而提高下载速度

### defer和async
prefetch和preload是解决下载时间问题, js执行也会阻断HTML Parser的, defer/async属性可以解决这问题, 可以结合webpack的插件:ScriptExtHtmlWebpackPlugin

defer是在所有元素解析完成后, DOMContentLoaded事件前执行js; js是依次执行
- 脚本的下载不会中断parse html
- 脚本执行完后才开始渲染(render/paint)

async是在脚本下载完成后执行, 执行顺序不固定, 用于加载google analysis之类的库, 谁快谁先执行
- 脚本的下载不会中断parse html
- 在render和paint后开始执行和求值, 这个过程在load事件前

## 拆包
要让js加载的更快的方法, 就是想办法让js包变到最小

一般我们可以使用BundleAnalyzerPlugin来分享webpack打包里大小, 找到比较大的包后, 就可以针对这个包做一些拆解

### 按需加载
将没有用到的代码, 就不要打进来了

```
import * as mathjs from 'mathjs’

vs

import {  create,  divideDependencies,  evaluateDependencies,  powDependencies
} from 'mathjs’

```

或者使用 LodashModuleReplacementPlugin 之类的拆包插件

### 替换
小包替换大包:

`Moment -> day.js  (Gzip 65k -> 2k) `

`Mathjs -> decimal.js  (Gzip 108k -> 12k)`


### 路由异步加载
```
// 借钱 - 申请
{
    path: '/loan',
    component: () => import(/* webpackChunkName: "loan.apply" */ '@/views/loan/apply.vue'),
},
// 借钱 - 绑卡
{
    path: '/loan/bindcard',
    component: () =>
      import(/* webpackChunkName: "loan.bindcard" */ '@/views/loan/apply-bind-card.vue'),
}

```

### yarn.lock文件导致重复打包
分析打包后的结构图, 你会或多或少的发现, 某个包因为版本不一致重复引用的情况:

比方A依赖=>B@0.0.1, C依赖=>B@0.0.3, 结果, 我们的vender.js可能即包括B@0.0.1, 也包括B@0.0.3; 当然, 你确认没问题后, 手动安装一个版本就行

### 还不满足? 上Performance指标
接下来, 我们可以尝试精简业务逻辑, 看看到底是哪个代码执行这么久, 然后干掉他

我们在开发环境下, 打开控制台, 找到Performance, 具体怎么用, 网上有介绍, 不多说, 看图:

<img src="http://ww1.sinaimg.cn/mw690/ba0e41a3ly1gbn0unwadsj21za184kjl.jpg" width="600px"/>

通过call tree, 你可以看到每个代码具体的执行时间(Total Time), 重耗时最长的开始, 一步一步的查下去, 你会发现, 原来有些不必要的代码, 执行了这么久.... 


## 利用缓存

这里又有很多话题, 但这这里简单提几点

- 接CDN, 大大的加快了静态资源的下周速度, 对于https的原则, 还避免了加密造成的性能损耗
- 离线缓存, 如果你是使用Hybrid技术实现的端内app, 可以尝试他, 能真正让你的应用秒开. 当然, 需要端上支持. 原理就是我们的静态文件不是用户访问才去下载, 而是伴随着发版, app帮我们预先加载到disk, 但需要商量好缓存的更新时机

## 图片的优化

最后一步, 就是当页面达到可交互状态时, 这个图片不一定都加载完了, 往往很多图片又是高清图片, 你又不能压缩的太厉害, 那这么办呢?



### 图片渐进加载
这里已经是可以一定程度上提升用户体验了

这里的目标就是不让用户看的图片加载是一卡一卡的, 而是平滑的加载. 并且如果判断浏览器是否支持webp, 支持的话, 使用webp

你可以封装成一个directive或者component, 类似

```
<img v-webp  src="//xxx.jpg"/>
<div v-webp />
```

img标签里的图片先是一张缩略图, 这个图片小, 很快就出来了, 与此同时, 加载高清图, 高清图片加载完成后, 再将缩略图替换

```js
// 部分实现
this.bigImg = new Image();
this.bigImg.onload = () => {
  this.isFullLoad = true;
};
```

 div元素如果使用了background: url(...)做背景图, 也可以加渐变效果

至于实现, 再说

### 图片懒加载

看情况, 如果图片特别长,特别多, 可以尝试. 原理就是监控图片是否在可视区域, 是的话才去加载


## 引用

- [js的加载速度](https://segmentfault.com/a/1190000011577248)
- [如何监控性能指标](https://zhuanlan.zhihu.com/p/82981365)
- [浏览器术语](https://mp.weixin.qq.com/s/GIpmZIY6yxGRBpkTDHuJuw)