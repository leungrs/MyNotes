# Node.js简介



### 简介

- 开源跨平台的JS运行时环境
- 使用V8作为引擎，V8是Chrome浏览器的内核，性能优异
- Node.js运行在单进程中，在单线程中处理每个请求，它的标准库里提供了一系列非阻塞IO原语，来让JS代码异步地执行。Node.js的许多库都是使用非阻塞IO，所以非阻塞是Node.js的天生具有的能力，阻塞反而是异常少见的。
- 天然的非阻塞使得Node.js无需使用多线程，就能够同时处理成千上万的并发请求。多线程并发比较容易出现bug。
- Node.js能够紧跟ES标准，你可以使用新的Node.js版本来支持新的ES标准。不像浏览器，你必须等待他们更新才能支持新的ES标准。
- Node.js生态完善，npm生态中有超过100W的第三方库可以自由使用。

### 历史

- 2009年，Node.js诞生
- 2010年，Express, Socket.io诞生
- 2011年，npm发布1.0版本，大公司开始采用Node.js，hapi诞生
- 2013年，使用Node.js的第一个大型博客平台Ghost诞生，Koa诞生
- 2014年，产生Node.js的一个大的fork，io.js，用来支持ES6
- 2015年，Node.js基金会诞生，io.js有合入到Node.js中
- 2016年，Yarn诞生，Node.js 6发布
- ...
- 2021年，Node.js 16发布



### 安装

- 可以使用nvm来安装和切换使用不同的Node.js版本



### 学习Node.js最好具备如下的JavaScript知识

#### 基础知识

- 词法结构
- 表达式，类型，类，变量，函数
- this
- 箭头函数
- 循环
- 作用域
- 数组
- 模板字面量
- 分号
- 严格模式
- ECMAScript 6, 2016, 2017

#### 异步编程知识

异步编程是Node.js的基础，所以你应该对如下概念及知识有所了解

- 异步编程和回调(Asynchronous Programming and Callbacks)
- 定时器(Timer)
- Promises
- Async和Await
- 闭包(Closure)
- 事件循环(The Event Loop)



### Node.js和浏览器的区别和联系

- 他们两者都使用JavaScript作为编程语言
- 在浏览器中，你大部分时间在使用DOM API或者其他的Web平台API，比如Cookies。Node.js中不存在这些，因为没有document, window对象。
- 在浏览器中，不能使用Node.js中由标准模块提供的一些功能，比如文件系统访问。
- 使用Node.js，是你在控制运行时环境，你可以指定你的程序依赖哪个Node.js版本。对于浏览器，你不能要求你的客户使用具体的某个浏览器。
- JavaScript还在发展，在Node.js中，你可以使用ES 6,7,8,9这些新的标准。浏览器中，一般就不能直接使用，可能需要Babel来进行转换，一般转成ES5的版本。
- 在Node.js中早就可以直接使用CommonJS模块系统，而浏览器中，ES模块系统的标准才刚开始形成，慢慢的被各大厂商实现。



### V8引擎

- V8是Chrome浏览器中JS引擎的名字。
- V8引擎提供的是一个运行JS代码的环境，它是独立于浏览器的。浏览器集成了V8来执行JS代码，此外浏览器提供了DOM API和Web Platform API。
- Node.js也采用V8作为的JS引擎。
- Node.js生态巨大，都归功于V8，Node.js不仅可以用于服务器，而且通过Electron等项目，Node.js也能用于桌面端程序。
- JS引擎，除了V8外，还有许多，比如Firfox的SpiderMonkey, Safari的JavaScriptCore（也就Nitro)等等。
- V8使用c++开发，持续进化，性能优异。
- JavaScript通常被认为是解释性的语言，但是现代的JS引擎，并不仅仅是解释执行JS代码，他们也对JS代码进行编译。V8就采用了即时编译技术（JIT）来加速JS代码的执行。