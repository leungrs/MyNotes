# 前端相应模块说明

### 底层支持模块

#### core-js

是很多polyfill的底层支持模块，如：babel-polyfill等。

### HTTP客户端

#### Axios

是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中。



### 测试相关模块

#### chai

是一个断言库，能够和许多的测试框架集成。支持浏览器和node。[参考](https://www.chaijs.com/).

可以是BDD（行为驱动开发）风格，也可以是TDD（测试驱动开发）风格。



### 代码转换和编译

#### Babel

是一个JavaScript的工具链，包含许多的工具，主要用来把ES6及以上的JavaScript版本转换成ES5等早期JavaScript版本代码。

提供：

- 语法转换
- 通过一个第三方ployfill库把目标环境中没有的JS特性给Polyfill上，使其具有该特性。
- 源码转换以及其它更多功能。

Babel可以通过插件，支持更多的功能。如：





#### Flow

Flow是一个给JS代码增加静态检查功能的npm模块，通过如下的方式进行：

- 写JS代码时，按照Flow的规范，给代码增加类型注解。
- Flow在后台进程中，对代码进行类型检查，报告相关错误。





### 上传

#### uppy



### UI库

#### element-ui

是一个基于vue2的UI库。



### 构建打包

#### Webpack

[Concepts | webpack](https://webpack.js.org/concepts/)

#### Rollup

[rollup.js (rollupjs.org)](https://rollupjs.org/guide/en/)

#### Parcel

[Parcel Documentation | Parcel 中文网 (parceljs.cn)](https://v2.parceljs.cn/docs/)

#### Browserify

[Browserify](https://browserify.org/)

#### esbuild

[esbuild - An extremely fast JavaScript bundler](https://esbuild.github.io/)

#### Vite

[Home | Vite (vitejs.dev)](https://vitejs.dev/)



### 调试

#### debug

一个小巧的 JS 调试工具，用来输出调试日志信息。[参考](https://github.com/visionmedia/debug)

主要功能：

日志格式化，控制具体哪个模块是否输出日志，日志输出目标的设置等



### 其它

#### throttle-debounce 

函数调用节流防抖



