---
layout: post
title:  "JFinal学习笔记-Demo 部署-lesson02"
date:   2014-07-29 22:18:24 +0800
categories: JFinal
tags: 
- JFinal
description: "JFinal 环境部署"
author: "acrens"
published: true
---
> 延续 JFinal lesson01，开始搭建一个 JFinal 案例
<!--more-->

### JFinal 是 JAVA 极速 WEB 开发的开源框架
一、环境：MyEclipse9+Tomcat6+MySql;
二、准备 jar 包，如下图：
![](/css/images/jfinal-jar.jpg)
三、开始在MyEclipse9+Tomcat6+MySql部署一个学生信息管理小Demo：

  1. 创建Web Project工程；
  2. 在WEB-INFO文件目录下新建lib文件夹并放入四个jar包，然后build path；
  3. 创建jfinalstudy01数据库并创建两张表student+classes，如下：
  ``` java
  create table classes
  (
      classesid      int(4) not null,
      classesname    varchar(40),
      classesaddress varchar(60)
  );
  create table student
  (
      studentid   int(4) not null,
      studentname varchar(10),
      studentage  int(4),
      studentsex  varchar(4),
      classesid   int(4)
  );
  ```
  4. 添加以下配置至web.xml（同时新建JFinalAllConfig类）:

  ![](/css/images/jfinal-allconfig.jpg)
  ``` java
  public class JFinalAllConfig extends JFinalConfig {

     @Override
     public void configConstant(Constants me) {
             me.setDevMode(true);
     }

     @Override
     public void configRoute(Routes me) {
             me.add("/", StudentController.class);
             me.add("/student", StudentController.class);
     }

     @Override
     public void configPlugin(Plugins me) {
             C3p0Plugin cp = new C3p0Plugin(
                             "jdbc:mysql://localhost:3306/JFinalStudy01", "root", "");
             me.add(cp);
             ActiveRecordPlugin arp = new ActiveRecordPlugin(cp);
             me.add(arp);
          
             // 三个参数依次为表名、主键、model
             arp.addMapping("student", "studentid", Student.class);
             arp.addMapping("classes", "classesid", Classes.class);
     }

     @Override
     public void configInterceptor(Interceptors me) {
             // TODO Auto-generated method stub

     }

     @Override
     public void configHandler(Handlers me) {
             // TODO Auto-generated method stub

     }

     }
  ```
  5. 新建Student、Classes的model：
    - Student：
    ``` java
    package com.phy.jfinal.model;

    import com.jfinal.plugin.activerecord.Model;

    public class Student extends Model {

           private static final long serialVersionUID = 1L;
        
           public static final Student dao = new Student();

           public Classes getClasses() {
                   return Classes.dao.findById(get("classesid"));
           }
    }
    ```
    - Classes：
    ``` java
    package com.phy.jfinal.model;

    import com.jfinal.plugin.activerecord.Model;

    public class Classes extends Model {
            
           private static final long serialVersionUID = 1L;
        
           public static final Classes dao = new Classes();
    }
    ```
  6. 新建StudentController（接收HTTP请求处理的控制器）：
  ``` java
  package com.phy.jfinal.controller;

  import java.util.List;

  import com.jfinal.aop.Before;
  import com.jfinal.core.Controller;
  import com.phy.jfinal.interceptor.StudentInterceptor;
  import com.phy.jfinal.model.Student;
  import com.phy.jfinal.validator.StudentValidator;

  public class StudentController extends Controller {
             private static int num = 0;

               // 进行获取学生信息列表前的拦截操作
             @Before(StudentInterceptor.class)
             public void index() {
                     List list = Student.dao.find("select * from student");
                     setAttr("studentList", list);
                     render("/index.html");
             }

             public void add() {
                     render("/add.html");
             }

             public void delete() {
                     // 获取url请求中第一个值
                     Student.dao.deleteById(getParaToInt());
                     forwardAction("/student");
             }

             public void update() {
                     Student student = getModel(Student.class);
                     student.update();
                     forwardAction("/student");
             }

             public void get() {
                     Student student = Student.dao.findById(getParaToInt());
                     setAttr("student", student);
                     render("/index2.html");
             }

            //StudentValidator是对进行保存操作时的变量的验证
           @Before(StudentValidator.class)
            public void save() {
                    Student student = getModel(Student.class);
                    student.set("studentid", num++).save();
                    forwardAction("/student");
            }
     }
  ```
  7. 拦截器及验证器：
    - 拦截器
    ``` java
    package com.phy.jfinal.interceptor;

    import com.jfinal.aop.Interceptor;
    import com.jfinal.core.ActionInvocation;

    public class StudentInterceptor implements Interceptor {

           @Override
           public void intercept(ActionInvocation ai) {
                   System.out.println("action注入之前");
                       ai.invoke();
                   System.out.println("action注入之后");
            }
    }
    ```
    - 验证器
    ``` java
    package com.phy.jfinal.validator;

    import com.jfinal.core.Controller;
    import com.jfinal.validate.Validator;

    public class StudentValidator extends Validator {

           //在校验失败时才会调用
           @Override
           protected void handleError(Controller controller) {
                   controller.keepPara("student.studentname");//将提交的值再传回页面以便保持原先输入的值
                     controller.render("/add.html");
           }

          @Override
          protected void validate(Controller controller) {
                   //验证表单域name，返回信息key,返回信息value
                   validateRequiredString("student.studentname", "studentnameMsg",
                           "请输入学生名称!");
          }
    }
    ```
  8. 前台实现（通过freemarker读取数据），在WebRoot目录下新建index.html，add.html，change.html(分别为学生信息列表获取、学生信息添加、学生信息修改，下面依次列出实现代码)：

    - index.html
    ![](/css/images/jfinal-html-index1.jpg)
    ![](/css/images/jfinal-html-index2.jpg)
    - add.html
    ![](/css/images/jfinal-html-add1.jpg)
    ![](/css/images/jfinal-html-add2.jpg)
    - change.html
    ![](/css/images/jfinal-html-change1.jpg)
    ![](/css/images/jfinal-html-change2.jpg)
  9. 三个CSS文件（添加在WebRoot目录下的css文件夹下：

    - common.css
    ![](/css/images/jfinal-css-common.jpg)
    - add.css
    ![](/css/images/jfinal-css-add.jpg)
    - index.css
    ![](/css/images/jfinal-css-index.jpg)
  10. 以下分别为整体项目目录结构及相关截图：

  ![](/css/images/jfinal-root.jpg)
  ![](/css/images/jfinal-page1.jpg)
  ![](/css/images/jfinal-page2.jpg)
  ![](/css/images/jfinal-page3.jpg)