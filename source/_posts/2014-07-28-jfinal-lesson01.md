---
layout: post
title:  "JFinal学习笔记-知识介绍-lesson01"
date:   2014-07-28 22:21:24 +0800
categories: JFinal
tags: 
- JFinal
description: "JFinal 知识介绍"
author: "acrens"
published: true
---
> JFinal 相关文章是个人刚毕业不久使用过的一个 Java 后台框架，感觉上手容易，搭建迅速，易于构建小型 Web 项目，以此对 JFinal 做个记录。
<!--more-->

### JFinal 是 Java 极速 Web 开发的开源框架
一、优点：入手快、学习花费少、开发迅速、代码量很少、易扩展、RESTful风格（扩展性好）。
二、缺点：比较适合于小型WEB应用开发。
三、特点：
1. MVC 架构、设计精巧;
1. 遵循 COC 原则、零配置、无附带xml文件;
1. 独创Db + Record模式、灵活便利;
1. ActiveRecord 支持，使数据库开发极致快速;
1. 自动加载修改后的 java 文件，开发过程中无需重启 web server(如：tomcat);
1. AOP 支持，拦截器配置灵活，功能强大;
1. Plugin体系结构，扩展性强;
1. 多视图支持，支持 FreeMarker、JSP、Velocity;
1. 强大的 Validator 后端校验功能;
1. 功能齐全，拥有 struts2 绝大部分核心功能;
1. 体积小仅 218K，且无第三方依赖。

四、体系架构
![JFinal 架构图](/css/images/jfinal.png)