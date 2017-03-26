---
layout: post
title:  "line-height 取值方式"
date:   2017-03-11 17:30:00 +0800
categories: line-height
tags: 
- css
- line-height
description: "line-height 取值方式"
author: "acrens"
published: true
---

> 相信很多前端er都使用过 line-height 来设置行高布局。下面看看官方定义：On block level elements, the line-height property specifies the minimum height of line boxes within the element.On non-replaced inline elements, line-height specifies the height that is used to calculate line box height.
<!-- more -->

### line-height 取值方式
有时候我们在开发的时候并没有太深入的去了解一个属性，比如：line-height 的不同取值达到的效果并不同，下面依次看看常用的几种取值方式。

----------

#### px 单位取值
line-height: 26px 目的就是直接定义目标元素的行高为 26px 的高度。

----------

#### % 百分比取值
line-height: 150% 一般用该方式定义目标元素的行高会配合 font-size: 14px 属性使用，因为用百分比当前元素的行高为 1.5 * 14px = 21px。且如果其层叠子元素没有定义 line-height 属性，不管有没有定义 font-size 属性，其层叠子元素行高均为 21px（与自身的 font-size 没有任何关系）。

----------

#### 倍数取值
line-height:1.5 用该方式一般也是配合 font:14px 属性使用，但是对层叠子元素的影响效果并不同，如果层叠子元素没有定义 line-height 属性，但是定义了 font-size 属性，那这些层叠子元素的行高为继承过来的 line-height 倍数值乘以自身的 font-size 属性。

----------

#### 总结
一般情况下，为了更加方便及清晰的使用 line-height，使用倍数取值是最佳的设置方式，如：body {font: 12px/1.5 tahoma,arial,"\5b8b\4f53"}。