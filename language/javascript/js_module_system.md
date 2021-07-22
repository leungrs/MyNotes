#  JS的模块系统

本文简单介绍如下几种常用的JS模块化方法，他们是：

1. CJS：CommonJS，node使用，是同步导入的
2. EMS：Ecmascript Module System
3. AMD:  Asyncronous Module Definition异步模块定义，
4. UMD:

### CommonJS 模块规范

是nodejs定义js模块化规范，node就是使用这样的js模块。这种模块不能直接在浏览器中使用，如果需要在浏览器中使用，需要转换一下。模块导入是同步的。

#### 定义模块

每个js文件就是一个模块，模块exports的内容就是模块暴露的接口。

```javascript
// exporting
module.exports = function doSomething(n) {
  // do something
}
```

#### 导入模块

使用require方法导入一个模块。

```javascript
// 从本地文件导入一个模块
const doSomething = require('./doSomething.js'); 
// 从node_modules导入一个模块
const path = require('path')
```

### AMD

异步模块定义，是一个规范。

#### 定义模块

使用define函数

```javascript
define('SomeMoudle', ['dep1', 'dep2'], function(dep1, dep2){
    
});
```



