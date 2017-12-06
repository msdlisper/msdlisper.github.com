---
layout: post
title:  "内联元素的一些总结"
date:   2017-08-10 19:55:21
categories: learn
tags: problem css
---

* content
{:toc}




# 前言
想研究下line-height, vertical-align, 因为平时项目经常用, 有时侯用的比较糊涂, 找到文档细细研究

## 总结
- 块状元素内部的内联元素的行为表现， 内联元素会在行上, vertical-align: baseline, 而baseline是会根据line-height来计算的

## line-height
- 要作用于inline formatting contexts上(外面是block, 里面是inline)
[示例](http://jsbin.com/vivewamusu/1/edit?html,output)


## vertical-align
- 用在inline-level elements: inline, inline-block, inline-table
- 对于内联元素
    - 有一个叫line box的东西, 这里有vertical-align, 里面有 baseline, text box(font-size), top/bottom edge(行高)
- vertical-align: 
    - baseline: 默认的, 等于line box的baseline
    - sub 在baseline下
    - super 在baseline上
    - <percetage>, 百分比, 将baseline设置相对于行高的值
    - <px>, 像素, 也是相对于行高
    - middle, 将元素的(上下边缘)中心对齐baseline, 并加上x的一半高度
        - [研究](http://jsbin.com/xeduqutedu/3/edit?html,output)
    - text-top, 将元素的上边缘对齐到元素text-box的上边缘
    - text-bottom, 将元素的上边缘对齐到元素text-box的下边缘
    - top,将元素的上边缘对齐到元素edge的上边缘
    - bottom,将元素的上边缘对齐到元素edge的上边缘
- 想让div里img居中, 其实用middle并不能绝对居中, 见[示例](http://jsbin.com/xeduqutedu/9/edit?html,output) 因为middle的对齐方式
- 解决: 因为middle是将元素中点对齐到x的baseline再加上x高度的一半, baseline加上高度的一半这个位置不在中点, 只要把字体大小设为0, 就在这一中点上 了
## inline-block
- 让内联元素可以设宽高, 并保留其它内联属性
- 默认他会以baseline对齐, 设置vertical-align: top可以顶部对齐
- 关于inline-block的base-line
[文档](https://www.w3.org/TR/CSS22/visudet.html#line-height)

The baseline of an 'inline-block' is the baseline of its last line box in the normal flow, unless it has either no in-flow line boxes or if its 'overflow' property has a computed value other than 'visible', in which case the baseline is the bottom margin edge.
[示例](http://jsbin.com/vivewamusu/7/edit?html,output)
[有三条线的示例](http://jsbin.com/xisiwiqewa/1/edit?html,output)

## 间隙问题
- 内联元素之间的空格会导致水平的间隙 [示例](http://jsbin.com/xeduqutedu/2/edit?html,output)
- 块级里面有内联元素, 如果用默认的baseline来对齐 baseline和edge下边缘有空隙[示例](http://jsbin.com/xeduqutedu/1/edit?html,output)
- 解决:
    - 让内联元素display: block
    - 让内联元素vertical-align: bottom/middle/top
    - 在块级元素上修改line-height
    - 用font-size: 0来间接设置line-height,line-height是一个相对单位(相对于内联元素), font-size为0, line-height也为0
[示例](http://jsbin.com/vivewamusu/2/edit?html,output)

## 居中对齐
- 一个div里的字体怎么上下居中?
    
    block元素设inline-height, inline不设置
    (最好也把inline-height = height 设height的目的是保证内联元素为空是,高度也撑开)
    不会绝对对齐, 应为inline元素会以baseline为齐, baseline并不可靠 (ant mobile是这么干的[ant](https://mobile.ant.design/components/button-cn/))
    [示例](http://jsbin.com/nubeviwasa/3/edit?html,output)

    baseline不可靠, 可以让它变的可靠: block元素font-size: 0; 让baseline, text-top, text-bottom重合, 并在block元素的中央
    优点: 不用设多余的margin,padding, 以后字变大点, div变高点都不用修改其它地方
    缺点是: div内的所有字体要手动设置
    [示例](http://jsbin.com/nubeviwasa/4/edit?html,output)

 

## 最后 Vertical-Align Demystified
- 如果在内联元素里遇到问题, 来问两个问题
     - Where is the baseline and top and bottom edge of the line box?
     - Where is the baseline and top and bottom edge of the inline-level elements?
    
# 引用 
- [w3c](https://www.w3.org/TR/CSS22/visudet.html#line-height)
- [css深入理解](http://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)
- [vertical-align](http://christopheraue.net/2014/03/05/vertical-align/)
- [font-size对行高的影响](https://www.w3cplus.com/css/css-font-metrics-line-height-and-vertical-align.html)