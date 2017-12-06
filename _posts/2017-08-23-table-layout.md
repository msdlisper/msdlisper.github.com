---
layout: post
title:  "表格布局算法"
date:   2017-08-10 19:55:21
categories: learn
tags: css problem
---

* content
{:toc}



# 引子

由于项目需要对table进行特别处理, 这里就研究下table的布局方式

# 两种算法

### table-layout: auto
[算法](https://www.w3.org/TR/CSS22/tables.html#auto-table-layout)
- 计算每个单元格 minimum content width (MCW): 如果 width 的值大于MCW, MCW = width, 要选这一列最大的MCW
- 计算每个单元格 maximum content width: 保证不内容不换行的宽度, 也是选这一列最大的值

示例:
[能让表格字段适应表格宽度](http://jsbin.com/yoqamit/4/edit?html,output)
#### 在td内直接插入文本
- 每个td设置ellipsis
- 并且加上 max-width, 就可以自动缩进,  
[例子](http://jsbin.com/tedocugoza/edit?html,output)
#### 在td内先插入div的情况
- 每个div ellipsis, 并设一个 max-width, 这样是不能自动根据宽度来省略的, 但table-layout: fixed可以
- [示例](http://jsbin.com/tedocugoza/1/edit?html,output)

### table-layout: fixed
[算法](https://www.w3.org/TR/CSS22/tables.html#fixed-table-layout)
- 如果一列元素没有设width的值, 那width会设为'auto'
- 第一行的列的宽度决定整行列的宽度, 单元格跨多列被平分
- 其它width=auto的列会被平分, 填满表格空余的地方
- 这个表格最终的宽度取table元素的width属性 与 所有列width之和 中的最大值

用它来实现一些效果:
- [比如想让table某些列的省略能适应表格的宽度](http://jsbin.com/yoqamit/3/edit?html,output)

#### 在td内先插入div的情况
- 每个div ellipsis, 并设一个 max-width, 也能自动根据宽度适应, 尽可能多的显示内容
- [示例](http://jsbin.com/tedocugoza/edit?html,output)


问题: 子行设width并不起作用, 他会根据第一行平分
# 参考

- https://developer.mozilla.org/en-US/docs/Web/CSS/table-layout
- https://www.w3.org/TR/CSS22/tables.html#width-layout
- https://csspod.com/table-width-algorithms/