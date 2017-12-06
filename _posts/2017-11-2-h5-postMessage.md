---
layout: post
title:  "H5 postMessage来实现跨域和跨窗口的通讯问题"
date:   2017-11-02 23:27:58
categories: learn 
tags:  跨域 h5
---

* content
{:toc}

记录postMessage的学习






## 需要解决的问题
在web开发时, 有时侯需要解决客户端的跨域问题
- 页面和其打开的新窗口的数据传递
- 多窗口之间消息传递
- 页面与嵌套的iframe消息传递


## 使用h5 提供的postMessage接口
[文档](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)
otherWindow.postMessage(message, targetOrigin, [transfer])

接收时用
```javascript
window.addEventListener('message',function(e){
                // if(e.source!=window.parent) return; // iframe的写法
                if(e.origin !== 'http://www.remote.com') return;
                // 搜到消息后回执
                e.source.postMessage("hi , remote message! ",
                                           e.origin);
            },false);
```


## 注意
- 如果是新窗口, postMessage必须是新窗口发出的, 然后在新窗口里监听
```js
var domain = "http://www.newOpen.com/";
var myPopup = window.open(domain 
            + 'path/html','myWindow');
myPopup.postMessage('message',domain);
```

- iframe要使用contentWindow
```js
var domain = "http://www.newOpen.com/";
var iframe = document.getElementById('myIFrame').contentWindow;
myPopup.postMessage('message',domain);
```
