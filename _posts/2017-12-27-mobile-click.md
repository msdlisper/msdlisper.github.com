---
layout: post
title:  "移动端的click"
date:   2017-12-27 21:47:58
categories: learn 
tags:  h5
---

* content
{:toc}

我想问移动端的点击事件有多坑..




## 引入
很happy, 不久前, 入坑移动端, 发现iso上点击有300ms延迟, 于是就网上找了下相关资料,发现了写问题:

## 300ms延迟, 你从哪里来?

让我们回到2007, iphone就要发布了, 苹果的工程师们为了让小屏幕的iphone能访问pc的网页, 聪明的决定: click事件加上300ms延迟来区分用户的双击缩放动作...

然后...各大手机浏览器厂商争相效仿...就成了现在的样子(是说老乔爱搞专利, 抄的人太多了)

这样带来的问题也非常明显, 那就是所有基于click交互的元素都受到影响, 大家的直观感受就是: 慢!!

因为100ms的延迟就会被人察觉

## 怎么解决

### 方案一 禁止缩放
- 代码

```
<meta name="viewport" content="user-scalable=no">
<!-- 或者 -->
<meta name="viewport" content="initial-scale=1, minimum-scale=1, maximum-scale=1">
```
- 原理: 上面的代码是禁止缩放, 延迟就是为了缩放, 那都没延迟了, 缩放也就没有必要了
- 支持情况: 在 Android 平台上，由 Chrome 最先提出，FireFox、Opera 等浏览器也相继支持；IOS 9.3 开始一度支持，**IOS10 开始不再支持**。
- safari不支持. 而且禁止缩放**会损害网站的可访问性(which may be an accessibility concern)**

### 方案二 视窗宽度设置为设备宽度
- 代码

```
<meta name="viewport" content="width=device-width">

```

- 原理: iPhone 诞生时就有的另一个约定是, 在渲染桌面端站点的时候, 使用 980 像素的视口宽度,而非设备本身的宽度(iPhone 是 320 像素宽), 上面代码告诉浏览器将视口大小设为设备本身的尺寸, 然后在 **Chrome 32 这一版中, 他们将在包含 width=device-width 或者置为比 viewport 值更小的页面上禁用双击缩放**, 没有必要双击, 就没有延迟了
- 支持情况: 自 Chrome 32 开始, FF,IE/Edge 也随后支持了; 2016 年 3 月,IOS **9.3** 开始支持。 

> 该方案的"禁止双击缩放"是遵守的[规则](https://trac.webkit.org/changeset/191644/webkit)
> - 当页面设置了视窗宽度且是初始尺寸(页面未缩放), 双击缩放才是被禁止的
> - 为了让用户结束后仍然能fast-click, 缩小时,只能缩小到初始尺寸, 不是最小尺寸

- 为什么不直接使用方案二?
    - 可以看看ant-disign的使用
        - 引入fastclick解决问题[576](https://github.com/ant-design/ant-design-mobile/issues/576#issuecomment-273065311)
        - iso 9.3以前的会有问题?毕竟iphone6s 15年9月发布, iso 9.3 16年3月发布, 用方案二还是有延迟
        - 代码
        
        ```
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no" />
      <script src="https://as.alipayobjects.com/g/component/fastclick/1.0.6/fastclick.js"></script>
        ```

### 方案三 指针事件
- 代码

```
button, .yourclass {
    -ms-touch-action: manipulation; /* IE10 */
    touch-action: manipulation; /* IE11 */
}
```

- 根据[w3c规范](https://w3c.github.io/pointerevents/#the-touch-action-css-property) [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action): touch-action的manipulation激活了平移和双指缩放手势, 而禁用了双击缩放等非标准的手势.
- [支持情况](https://caniuse.com/#search=touch-action): Opera Mini不支持, FF需要手动启动,iso上,**baiduhi_ios/6.7.1**不支持

### 方案四 [FastClick](https://github.com/ftlabs/fastclick)
- 代码

```
window.addEventListener( "load", function() {
    FastClick.attach( document.body ); // 直接绑定到 <body> 上可以确保整个应用都能受益
}, false );
```

- 原理: FastClick监听touchend事件, 通过[DOM自定义事件](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)立即触发一个模拟的click事件, 并把300ms之后的click事件阻止掉
- FastClick会[检测](https://github.com/ftlabs/fastclick/blob/master/lib/fastclick.js#L734)你是否用了其它方案, 避免冲突
- 缺点和已知问题:
    - fastclick是监听touchend, 但如果我再监听touchend, 比如列表滑动的实现, fastclick会误认为是一次点击, 我在android(目前所有机型)上有这个问题[178](https://github.com/ftlabs/fastclick/issues/178)
    - label和input 也不能正确的绑定[460](https://github.com/ftlabs/fastclick/issues/460)
    - github 上似乎很久没人搭理了..

### 最后使用的
- 本来想使用方案三, 但无奈ios的百度浏览器不支持..
- 于是暂时先用4


### Safari在这个问题上
可以看出, 方案1,2,3都是ios不友好的. [2016 年 3 月发布的 IOS 9.3 移除了 300ms 延迟、从而实现了fast-tap](https://developer.apple.com/library/content/releasenotes/General/WhatsNewInSafari/Articles/Safari_9_1.html#//apple_ref/doc/uid/TP40014305-CH10-SW8), 但[IOS10 无视了禁用缩放(user-scalable)](https://developer.apple.com/library/content/releasenotes/General/WhatsNewInSafari/Articles/Safari_10_0.html#//apple_ref/doc/uid/TP40014305-CH11-SW1), 所有单靠\<meta\>来解决延迟, 不是很靠谱. 谁知道safari后面会有什么新的想法.

## 点击穿透
其实我还没有遇到过(没用过Zepto, 不知有此坑), 但这个问题和300ms有关

### 来历
PC上的点击是鼠标事件, 一次点击行为的触发过程是: mousedown ->  mouseup -> click

手机上一次点击是 touchstart  -> touchend --------------> mousedown -> click -> mouseup
也会响应鼠标事件, 只是有300ms延迟. 而且没有像PC端的click一样的事件(touchend并代替不了), 于是许多js库(Zepto, JQuery)就来模拟一个click, 叫**tap**事件, 并, 引出了*点击穿透*.

### 示例
- jsfidden

<script async src="//jsfiddle.net/msdlisper/xe0e3wpu/1/embed/"></script>

- codepen
    
<p data-height="310" data-theme-id="0" data-slug-hash="PEpGVe" data-default-tab="css,result" data-user="msdlisper" data-embed-version="2" data-pen-title="点击穿透" class="codepen">See the Pen <a href="https://codepen.io/msdlisper/pen/PEpGVe/">点击穿透</a> by msdlisper (<a href="https://codepen.io/msdlisper">@msdlisper</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

### 解释
- 结合Zepto源码, tap事件是通过监听document上的touch事件来完成tap的模拟, 是通过**事件冒泡**实现的, 但touchstart/touchend会触发click事件的, 因为click事件有延迟, 所以300ms后, 触发到屏幕上最顶上的元素的click事件
- 好,来理一下思路: 现有A,B两层, A在B上. 我点A -> 触发A touchstart -> touchend -> tap -> A(), 让A立马消失 ----300ms后---> click触发 -> 因为A不在了, 锅就这样甩给了B -> B.click() -> 完成了从点击A到穿透B的过程
    
### 解决
- 最直接的方法: fastclick, 让click延迟消失
- 用CSS3的 poiner-events: none, 元素不再是鼠标事件的目标, 对click不感冒
- 加一个中间层C, 让A消失后, C来背锅
    
## 参考

- [(2013)300 毫秒点击延迟的来龙去脉](https://thx.github.io/mobile/300ms-click-delay)
- [(2015)也来说说touch事件与点击穿透问题](https://segmentfault.com/a/1190000003848737)
- [(2016) 300ms-tap-delay-gone-away](https://developers.google.com/web/updates/2013/12/300ms-tap-delay-gone-away)
- [(2017)移动端 click 事件 300ms 延迟的前世今生](http://baishusama.github.io/2017/03/27/%E7%A7%BB%E5%8A%A8%E7%AB%AF-click-%E4%BA%8B%E4%BB%B6%E7%9A%84-300ms-%E5%BB%B6%E8%BF%9F/)



