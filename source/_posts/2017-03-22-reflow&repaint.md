---
layout: post
title:  "浏览器渲染那些事之 Reflow、Repaint"
date:   2017-03-22 22:09:00 +0800
categories: 浏览器渲染
tags: 
- Reflow
- Repaint
description: "Reflow、Repaint是什么，如何触发Reflow、Repaint，如何避免Reflow、Repaint"
author: "acrens"
published: true
---

> 在进行网页开发的时候，一般会忽略到页面渲染给浏览器带来的性能问题；在实际情况中，浏览器进行页面渲染会进行大量的计算，来确定每个可见元素在屏幕上的精确位置、大小，还需要将每个确定好的像素绘制到屏幕上，这些操作都需要消耗大量的资源；如果反复的进行这些操作，对用户设备性能损耗不容乐观，因此希望通过这篇文章加深大家对浏览器渲染过程的理解，并希望大家能够重视渲染过程带来的性能问题。
<!-- more -->

### 浏览器渲染那些事之 Reflow、Repaint

#### 浏览器内核（渲染引擎）
在各个浏览器厂商你追我赶的形势下，截止今日，产生了很多不同的浏览器，各个浏览器本质大同小异，核心部分基本相似，由渲染引擎和 JS 引擎组成。当你在访问网站页面的时候，浏览器做了很多事情，从发送请求，到解析 HTML 源码，构建渲染树，最后将内容像素绘制到设备屏幕上，一步步完成用户最终需要的场景。然而，在浏览器完成整个渲染的过程中，最为核心的就是“渲染引擎”。以下分别列出一些主流浏览器的渲染引擎：
- [chrome](http://www.google.cn/intl/zh-CN/chrome/browser/desktop/index.html) - webkit
- [safari](http://www.apple.com/cn/safari/) - webkit
- [opera](http://www.opera.com/zh-cn) - webkit（早期是 presto）
- [firefox](https://www.mozilla.org/en-US/firefox/products/) - gecko
- [ie](https://support.microsoft.com/zh-tw/help/17621/internet-explorer-downloads) - trident

#### 渲染流程
结合浏览器渲染原理，来剖析以下代码渲染构建过程：
1. HTML 源码：
``` html
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <link href="style.css" rel="stylesheet">
    <title>browser rendering</title>
  </head>
  <body>
    <p>Hello <span>web performance</span> students!</p>
    <div><img src="awesome-photo.jpg"></div>
  </body>
</html>
```
2. CSS 源码：
``` css
body { font-size: 16px }
p { font-weight: bold }
span { color: red }
p span { display: none }
img { float: right }
```
3. 该图为以上源代码构建树：
![构建渲染树](/css/images/render-tree-construction.png)

浏览器渲染页面整个过程描述：
1. 首先，解析 HTML Source，构建 DOM Tree；
2. 同时，解析 CSS Style，构建 CSSOM Tree；
3. 然后，组合 DOM Tree 与 CSSOM Tree，去除不可见元素，构建 Render Tree；
4. 再执行 Reflow，根据 Render Tree 计算每个可见元素的布局（几何属性）；
5. 最后，执行 Repaint，通过绘制流程，将每个像素渲染到屏幕上。

注意：
1. Render Tree 只包含渲染网页所需要的节点；
2. Reflow 过程是布局计算每个对象的精确位置和大小；
3. Repaint 过程则是将 Render Tree 的每个像素渲染到屏幕上。

#### 哪些阶段影响渲染效率
我们都知道，网页是需要挂载在客户端的浏览器上运行，但是随着互联网的快速发展，为保证用户访问应用的流畅性，往往对客户端的设备配置要求较高。然而，对于用户的设备，我们是无法控制；因此，作为一名开发者，就需要找到除了用户设备配置之外导致访问不流畅的问题，下面就从渲染流程中找到影响性能的问题。

浏览器初始化渲染时会执行一次 Reflow 过程，这个过程主要是用来确定页面上每个元素在屏幕上的几何位置属性。但是，每执行一次 Reflow 会需要花费大量的时间，耗费大量的设备资源，所以尽量避免执行 Reflow 过程。同时，执行完 Reflow 都会伴随着一次 Repaint 过程，这个过程也会耗费大量的计算机资源，严重影响页面的渲染性能。所以，在浏览器渲染阶段，主要影响页面渲染的阶段是 Reflow 和 Repaint 过程，因此在编写代码时应该尽量避免 Reflow 和 Repaint 过程的执行，如：避免在 JS 代码里直接改变元素的几何属性。

#### reflow & repaint 简介
1. reflow 在渲染过程中称为回流，发生在 Render Tree 阶段，它主要是用来确定每个元素在屏幕上的几何属性，需要大量计算每个元素的位置。在代码里每改变一个元素的几何属性，均会发生一次回流过程。

2. repaint 在渲染过程中称为重绘，发生在 reflow 过程之后，当元素的几何属性确定之后便要开始将元素绘制在屏幕上展示。repaint 执行过程就是将元素的颜色、背景等属性绘制出来。在代码里没改变一次元素的颜色等属性时均会对相关元素执行一次重绘。

#### 如何触发 reflow 和 repaint 过程
1. 改变元素 font-size：
``` javascript
document.getElementsByTagName('body')[0].style.fontSize = '20px'; // reflow,repaint
```
2. 改变元素盒模型margin、border、padding、width：
``` javascript
let styles = document.getElementsByTagName('body')[0].style;
styles.margin = '40px'; // reflow,repaint
styles.border = '40px solid #f00'; // reflow,repaint
styles.padding = '40px'; // reflow,repaint
styles.width = '300px'; // reflow,repaint
```
3. 改变元素颜色、背景色属性：
``` javascript
let styles = document.getElementsByTagName('body')[0].style;
styles.color = '#fff'; // repaint
styles.backgroundColor = '#f00'; // repaint
```
4. 特殊：offset*、scroll*、client*、getComputedStyle、currentStyle：
- 由于浏览器在处理批量修改页面元素样式时，会将批量操作缓存起来，然后再做一次 reflow 过程（异步 reflow），避免每次操作都执行 reflow 消耗资源。但是如果在某个操作之后立马调用了以上执行属性，为了等够得到最新的样式，会检查缓存的操作，是否需要 reflow，这样就 flush 出最新的样式。

#### 如何减少 reflow 和 repaint 过程
1. 减少 JS 逐行修改元素样式：
``` javascript
let body = document.getElementsByTagName('body')[0];
body.className += ' class-name';
```
2. 离线处理 DOM 操作：
    - 通过 documentFragment 集中处理临时操作；
    - 克隆节点进行操作，然后进行原节点替换；
    - 使用 display:none; 进行批量操作。
    
3. 减少样式的重新计算，即减少 offset*、scroll*、client*、getComputedStyle、currentStyle 的使用，因为每次调用都会刷新操作缓冲区，执行 reflow & repaint。

#### 参考资料
- [渲染性能](https://developers.google.com/web/fundamentals/performance/rendering/)；
- [Rendering: repaint, reflow/relayout, restyle](http://www.phpied.com/rendering-repaint-reflowrelayout-restyle/) - [译文](http://www.cnblogs.com/ihardcoder/p/3927709.html)；
- [浏览器的渲染原理简介](http://coolshell.cn/articles/9666.html)；
- [分析运行时性能](https://developers.google.com/web/tools/chrome-devtools/rendering-tools/)；
- [如何使用 Timeline 工具](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool#profile-js)。