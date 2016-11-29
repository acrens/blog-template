---
layout: post
title:  "来，加入前端自动化单元测试"
date:   2016-09-20 23:30:00 +0800
categories: 自动化单元测试
tags: 
- 单元测试
- auto-unit-test
description: "前端自动化单元测试"
author: "acrens"
published: true
---

> 最近闲来无事，开始摸索前端单元测试。一是不备之需，二是确实在实际项目中能够用到单元测试。这样可以提高开发效率，提升代码质量，完全可以单独对 JS 进行测试，无需页面，不依赖其他第三方。
<!-- more -->

### 为什么需要单元测试
在这里首先需要知道单元测试的目的及结果：
1. 使代码健壮，质量高，兼容各种临界点；
2. 减少 QA 测试报告的反馈，提高自我影响力；
3. 保证代码的整洁清晰。

如果需要刨根问底追究为什么需要进行单元测试，那我们可以开始讲讲实际项目开发中遇到的一些问题：
1. QA 不断反馈代码有 BUG （此时你正在投入的开发，然后被打扰...）；
2. 代码出现 BUG，叠加代码修复 BUG（代码越来越难维护...）;
3. 已经开发完成一个模块，但是没有页面提供调试测试；
4. 你开发完成的功能，每次都有许多细小的 BUG（个人影响力下降...）。

好了，列举了这么多原因，相信你也开始心虚了，回去继续搬砖检查检查代码有没有问题，如果你面色从容，大神，请手下我的膝盖。

总结：单元测试的目的只有一个，用来确定是否适合使用
### 如何进行单元测试
如果明白了为什么要进行单元测试，相信你已经可以开始着手为自己的代码写一些单元测试代码。测试从字面理解就是检验，看对象是否达标，达标就是 pass，不达标就是 fail。产品有这样一个需求，如果结果是 3 达到目标且返回的为有效的数字类型才可以进行比较，下面看个栗子：
``` javascript
/**
 * 获取 a 除以 b 的结果
 * @param  {[Number]} a [数字]
 * @param  {[Number]} b [数字]
 * @return {[Number]}   [结果数字]
 */
function division(a, b) {
    return a / b;
}

// 测试代码
function test() {
    var result = division(6, 2);
    
    if (result === 3) {
        console.log('pass');
    } else {
        console.log('fail');
    }
}
```
咋一看上面的代码没什么问题，可以满足产品的需求，但是问题来了，如果 b 为 0，这个模块就出现了 BUG，同时如果下次需要达到其他的值就算通过，那就得去修改测试代码，这样的测试代码本身也太不健全。于是乎有了下面的方式：
``` javascript
/**
 * 获取 a 除以 b 的结果
 * @param  {[Number]} a [数字]
 * @param  {[Number]} b [数字]
 * @return {[Number]}   [结果数字]
 */
function division(a, b) {

    if (b === 0) {
        return 0;
    } else {
        return a / b;
    }
}

function test(name, result, expect) {

    if (result === expect) {
        console.log(name + '-> pass');
    } else {
        console.log(name + '-> fail');
    }
}
test('normal number', division(6, 2), 3);
test('zero', division(6, 0), 0);
```
如果需要期望值为 10 就通过，那可以这样：
``` javascript
test('normal number is 10', division(20, 2), 10);
```
### 单元测试环境搭建及代码示例
但是随着前端迅速的发展，也出现了很多测试框架，下面我演示我在实际项目中使用的测试框架环境配置 karma + jasmine，对于 karma、jasmine 我就不介绍，网上一搜一大把介绍：
1. 安装 node 环境
依赖于 node 作为基础环境，安装完成在控制台运行下面命令查看是否安装成功。
``` javascript
node -v
```
2. 新建目录并通过以下命令初始化项目配置 package.json
``` javascript
npm init
```
    在 package.json scripts: {} 添加以下内容：
``` javascript
"test": "karma start karma.conf.js"
```
3. 依次安装测试框架
``` javascript
npm install karma -g
npm install jasmine --save-dev
npm install karma-jasmine --save-dev
npm install karma-chrome-launcher --save-dev
npm install jasmine-core --save-dev
```
    或者一次性安装
``` javascript
npm install karma -g
npm install jasmine karma-jasmine karma-chrome-launcher jasmine-core --save-dev
```
4. 运行以下命令新建 karma.conf.js（根目录下不是必须）
``` javascript
karma init
```
    文件内容及说明：
``` javascript
/**
 * karma 自动化测试参数配置
 */

module.exports = function(config) {
    config.set({

        // 基础路径，用在files，exclude属性上
        basePath: '',

        // 可用的测试框架: https://npmjs.org/browse/keyword/karma-adapter
        frameworks: ['jasmine'],

        // 需要加载到浏览器的文件列表
        files: [
            './src/**/*.js',
            './test/unit/specs/*.spec.js'
        ],

        // 排除的文件列表
        exclude: [
            'karma.conf.js'
        ],

        // 在浏览器使用之前处理匹配的文件
        // 可用的预处理: https://npmjs.org/browse/keyword/karma-preprocessor
        preprocessors: {},

        // 使用测试结果报告者
        // 可能的值: "dots", "progress"
        // 可用的报告者: https://npmjs.org/browse/keyword/karma-reporter
        reporters: ['progress'],

        // web server port
        port: 9876,

        // 启用或禁用输出报告或者日志中的颜色
        colors: true,

        /**
         * 日志等级
         * 可能的值：
         * config.LOG_DISABLE //不输出信息
         * config.LOG_ERROR    //只输出错误信息
         * config.LOG_WARN //只输出警告信息
         * config.LOG_INFO //输出全部信息
         * config.LOG_DEBUG //输出调试信息
         */
        // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
        logLevel: config.LOG_INFO,

        // 启用或禁用自动检测文件变化进行测试
        autoWatch: true,

        // 测试启动的浏览器
        // 可用的浏览器: https://npmjs.org/browse/keyword/karma-launcher
        browsers: ['Chrome'],

        // 开启或禁用持续集成模式
        // 设置为true, Karma将打开浏览器，执行测试并最后退出
        singleRun: false,

        // 并发级别（启动的浏览器数）
        concurrency: Infinity,

        // 依赖插件
        plugins: [
            'karma-chrome-launcher',
            'karma-jasmine'
        ]
    })
}
```
5. 新建源代码及测试代码目录，目录结构如下：
``` javascript
project
    - node_modules
        - *(node 模块)
    - src
        - FQA
            - index.js
    - test
        - unit
            - specs
                - *.spec.js
    - karma.conf.js
    - package.json
```
6. 测试代码
    + index.js 源码
    ``` javascript
    /**
         - test map method callback and parseInt param use
         - @return {[Array]} [Array]
         */
        function checkMap() {
            var nums = ['1', '2', '3'];
        
            return nums.map(parseInt);
        }
        
        /**
         - test null is Object，and common object is same
         - @return {[Array]} [Array]
         */
        function typeofAndInstanceOf() {
            var result = [];
            result.push(typeof null);
            result.push(null instanceof Object);
        
            return result;
        }
        
        /**
         - 检测操作符优先级
         - @return {[string]} [返回字符串]
         */
        function checkOperators() {
            var result = 'autoTest';
            result = 'Value is ' + (result === 'autoTest') ? 'Something' : 'Nothing';
        
            return result;
        }
    ```
    + fqa.spec.js 测试代码
    ``` javascript
        /**
         - test index.js checkMap method
         - detail:
         -     parseInt(val, base), base is 2 ~ 36, otherwise value equal NaN.
         */
        describe('test map and callback parseInt', function() {
            
            it('a array call map', function() {
                var nums = checkMap();
                console.log(nums);
        
                expect([1, NaN, NaN]).toEqual(nums);
            });
        });
        
        /**
         - test index.js typeofAndInstanceOf method
         - detail:
         -     typeof null qeual 'object', but null instanceof Object equal false, because null Constructor not Object.
         */
        describe('test null is object', function() {
            
            it('null object', function() {
                var result = typeofAndInstanceOf();
                console.log(result);
        
                expect(['object', false]).toEqual(result);
            });
        });
        
        /**
         - test index.js checkOperators method
         - detail:
         -     compare operator precedence, + gt ?.
         */
        describe('test null is object', function() {
        
            it('test operator preceence', function() {
                var result = checkOperators();
                console.log(result);
        
                expect('Something').toEqual(result);
            });
        });
    ```
7. 运行 sudo npm run test 执行测试代码
``` javascript
"scripts": {
    "test": "karma start karma.conf.js"
}
```
8. 结果：
![运行结果](/css/images/result.png)

#### 解答
1. npm run test 运行的实际上是 package.json 中配置的命令：
``` javascript
"test": "karma start karma.conf.js"
```
2. describe 定义测试模块，it 测试一个单元，describe 内部可以同时定义多个 it，因此可以做一系列的单元测试，测试方法详见[官方文档](http://jasmine.github.io/edge/introduction.html)。
3. karma.conf.js 配置 files 设置测试时需要被加载的文件
``` javascript
files: [
    './src/**/*.js',
    './test/unit/specs/*.spec.js'
]
```
#### 总结
希望看完这篇文章，你也能够动起手来，开始编写一些单元测试代码，提高代码的质量，提升自己的周围影响力。本篇文章内容表述了实际项目开发中会遇到的问题，我们可以通过单元测试来减少这类问题的发生，以提高代码的安全性，代码的质量，从而保证产品的稳定性。