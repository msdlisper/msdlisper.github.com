---
layout: post
title:  "浏览器页面重排 与 requestAnimationFrame"
date:   2017-10-15 17:55:21
categories: learn
tags: webApi problem
---

* content
{:toc}




## 手机百度浏览器页面闪动问题
- 原因: style-loader在unuse style时组件并没有卸载, 造成了页面多余的flow
- 解决: 理解页面渲染, 避免不必要的flow

## 怎么解决?
- 通过requestAnimationFrame api来控制repaint的时机
- 这个是https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame上面的例子, 我稍微改了下

```
var start = null;
var element = document.getElementById('Specification');
element.style.position = 'absolute';

function step(timestamp) {
  if (!start) start = timestamp;
  var progress = timestamp - start;
  element.style.left = 200 + 'px';
  console.log(progress)
 
}
window.requestAnimationFrame(step);
```

如果没有window.requestAnimationFrame, 到element.style.position = 'absolute';这一句就会重绘, 但上面这么写, 会先element.style.left = 200 + 'px';然后才是element.style.position = 'absolute'; 然后才是重绘 

如果想在下一帧继续执行另一个动画, 需要在回调里继续调用requestAnimationFrame, 有点像flash的动画脚本, 保证从这一帧到下一帧之间不做多余的渲染

文档里是这么说的

```
the window.requestAnimationFrame() method tells the browser that you wish to perform an animation and requests that the browser call a specified function to update an animation before the next repaint. The method takes a callback as an argument to be invoked before the repaint.
```

- 通过fastdom来提高性能
    + 提高排版速度
        * http://wilsonpage.github.io/fastdom/examples/aspect-ratio.html
        * requestAnimationFrame 可以做到这一帧做好所有的读操作, 下一帧统一做写的操作(调用两次requestAnimation)
        * withoutFastDom 可以做到在前一帧将一系列的 读-写 操作坐完, 然后再执行下一帧: (Each measure/mutate job is added to a corresponding measure/mutate queue. The queues are emptied (reads, then writes) at the turn of the next frame using window.requestAnimationFrame.)(调用一次requestAnimation)
        * withoutFastDom 读写在一起同步执行, 是最慢的
    + 提高兼容性

## 引用
- https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame
- https://github.com/wilsonpage/fastdom
- https://mozillagfx.wordp…-part-1-browsers-today/#
