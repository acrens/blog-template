---
layout: post
title:  "XSS 防御之战"
date:   2016-09-11 22:34:00 +0800
categories: XSS
tags: 
- XSS
description: "XSS 防御之战"
author: "acrens"
published: true
---

> 写这篇文章主要是为了记录下最近项目开发中遇到的一个问题及解决过程；XSS 相信大家都知道，其相关知识就不在这里多讲；本篇文章主要侧重前端在获得后端给予的数据时是如何防御 XSS。 看到这里应该有人会说，后端都已经给你进行了 XSS 过滤，前端还需要做这些干什么，我的答案是：永远不要相信别人给你的东西是和你想要的 100% 一致。
<!--more-->

### 防御 XSS 攻击
### 项目场景
1. 前端基于 RESTful 风格 API 异步获取后端数据；
2. 数据渲染都是通过 underscore template 渲染页面。

### 解决方案
封装一个过滤特殊字符【如：<、>、%、@等】全局方法，供渲染数据之前调用。

但是发现项目是通过 underscore template 渲染页面的，想想可以在这里做点手脚；经过一番调研，果真如此，template 提供两种变量获取数据的方式，分别为：<%= variable %>、<%- variable %>。
1. <%= variable %> 不对数据做任何处理；
2. <%- variable %> 对数据进行 HTML 转义（即特殊字符转义）。

经过多次尝试，现在有四种选择方案：
1. <%= variable %> & html(str)
存在问题：如果后台未进行 XSS 防御，前台过滤无效：
``` javascript
$('.J-content').html(_.template('<%=content%>', {content: '<script>alert(1);</script>'}));
```
2.  <%= variable %> & text( str)
存在问题：如果后台进行了 XSS 防御，前台展示数据未还原：
``` javascript
$('.J-content').text(_.template('<%=content%>', {content: '&lt;script&gt;alert(1);&lt;/script&gt;'}));
```
3.   <%- variable %> & html(str)
存在问题：如果后台进行了 XSS 防御，前台展示数据未还原：
``` javascript
$('.J-content').html(_.template('<%-content%>', {content: '&lt;script&gt;alert(1);&lt;/script&gt;'}));
```
4.   <%- variable %> & text(str)
存在问题：不管后台进行还是没进行 XSS 防御，前台展示数据均未还原：
``` javascript
$('.J-content').text(_.template('<%-content%>', {content: '&lt;script&gt;alert(1);&lt;/script&gt;'}));

$('.J-content').text(_.template('<%-content%>', {content: '<script>alert(1);</script>'}));
```

灵机一动的我细心一想上述四种方法不可能不行，后台找到了原因所在，是因为 html 和 text 方法的区别，html 方法渲染页面会讲转义字符转换为 HTML 代码，text 方法渲染页面只是纯粹的内容替换，但是 html 方法只能将 &amp 渲染成单独的 &，所以通过 <%- variable %> 转义 &lt 成 &amp\;lt 之后再由 html 方法渲染在页面显示为 &lt，所以通过 replace 方法替换所有 &amp，如下代码：
``` javascript
$('.J-content').html(_.template('<%-content%>', {content: '&lt;script&gt;alert(1);&lt;/script&gt;'}).replace(/&amp;/g, '&'));
```