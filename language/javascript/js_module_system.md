#  JS的模块系统



### 背景

早期的JavaScript程序主要是为了完成Web页面的简单的交互效果，所以代码量很小，模块化也没有多大的意义。但是现在出现了使用JavaScript实现的，在前端浏览器中运行的完整的应用，代码量也越来越大。特别是出现了Nodejs，使用JavaScript的地方越来越多。此时就很有必要提供一种机制，能够把JavaScript代码分割成一些列的模块，在需要的地方直接导入。



NodeJS已经能够使用模块很长时间了。同时也出现了很多的库和框架来支持编写模块化的JavaScript代码，比如基于CommonJS和AMD规范的模块系统RequireJS，以及最近出现的Webpack，Babel等。幸运的是，JavaScript从ES 6开始原生地支持模块化了，这种原生的模块化也得到了越来越多的现代浏览器支持，这肯定是个好事情，因为相比通过库或者框架等方式来支持模块化的方式，浏览器可以优化模块的加载，从而提升效率。



本文将简单介绍这几种JavaScript的模块化方法，包括NodeJS的模块化，以及通过库或者框架实现的模块化，当然也包括原生的模块化。

1. CJS：CommonJS，node使用，是同步导入的。
2. EMS：ECMAScript Module System，原生的模块定义，从ES6开始支持，通过import和export实现。
3. AMD:  Asyncronous Module Definition异步模块定义，通过define，require实现
4. UMD：Universal Module Definition，统一的模块定义。在以上三种情况下，都能使用的模块定义。



### EMS：原生的模块化

JavaScript原生的模块化，是在ES6中引入的，通过```import```和```export```语句来提供支持的。一个js文件就是一个JS模块，也叫ES 模块，或者ECMAScript模块。

只能在模块文件中使用export和import语句，常规js文件中，不能使用这两个语句。

#### .mjs 还是 .js ？

模块的后缀名一般为.mjs或者.js，V8推荐使用.mjs，因为：

- 通过后缀名可以清楚地判断，哪些文件是普通的js代码文件，还是一个模块代码文件。
- 能够确保让NodeJS或者其他的构建工具如Babel，在解析JS文件时，把它当做一个模块文件解析。



#### export 语句

可以通过export导出的对象有：functions，var， let， const，class，这些对象，必须是模块文件中的顶层对象。你不能导出一个在函数中定义的对象。

```javascript
/* module_a.js */
// 逐个导出
export let PI = 3.14;
export var name = "Winson";
export const book = "Alice Dream";
export function add(a, b) {
    return a + b;
}
export class Animal {...}

// 也可以在模块文件的末尾，统一导出
let PI = 3.14;
var name = "Winson";
const book = "Alice Dream";
function add(a, b) {
    return a + b;
}
class Animal {...}
export {PI, name, book, add, Animal}
```

##### 命名导出

```javascript
// 命名导出, 就是在正常的对象声明语句前，加上export
export var a = 5;
export let b;

// 命名导出后，导入就必须和导出的名字相同
import {a, b} from 'xxx.js'
```



##### 默认导出

只能有一个默认导出。

```javascript
// 默认导出，就是导出时加上关键字default
// 它表示当前模块默认导出的是哪个对象
// 假设如下导出包含在a.js中
export default function(a, b) {
    return a + b;
}

// 当然加上名字也没问题
export default function add(a, b) {
    return a + b;
}

```



#### import 语句

可以在一个模块中通过import语句导入另一个模块中export的对象。

导入时的限制：

- 指定一个模块的路径只能是URL形式，包括相对URL和绝对URL，如：```./```, ```../```, ```/```。
- 只能在模块的顶层导入另一个模块，不能在函数中导入，如果要实现在函数中动态导入，可以使用import()。

如：

```javascript
/* module_b.js */

// 如下都是合法的导入
import {PI, name, book, add, Animal} from "./module_a.js"
import {shout} from './lib.mjs';
import {shout} from '../lib.mjs';
import {shout} from '/modules/lib.mjs';
import {shout} from 'https://simple.example/modules/lib.mjs';

// 如下的导入还不支持
import {shout} from 'jquery';
import {shout} from 'lib.mjs';
import {shout} from 'modules/lib.mjs';
```



#### 有关模块和常规JS文件的其他区别

- 模块文件，不能通过本地文件引入（如：file://），否则会报CORS错误，只能通过服务器来测试。
- 模块中的代码，都是严格模式。
- 通过script标签引入模块，不需要使用defer，因为默认就是defer了。
- 模块只执行一次，即使引入很多次。
- 模块有自己的作用域，在外部不能访问，也就不会污染全局环境。
- 模块中的this是undefined，模块中使用globalThis才能引用全局对象。
- 模块中不能使用HMTL风格的注释了，如：<!-- TODO: xxx
- 模块中可以使用顶层的await关键字，常规JS文件中不行，常规JS文件顶层使用await，只是把它当做一个变量名，没有实际的含义，常规JS只能在async函数内部使用await关键字。



由于存在这些不同，所以JavaScript运行时，需要知道哪些文件是模块，哪些不是。



####  在HTML中使用模块

##### type=module

```html
<!-- 使用type=module，标识main.js是一个模块，这样浏览器就能识别 -->
<script type="module" src="module_a.js"></script>
<script type="module">
	import {A, b, c} from './module_c.js'
</script>
```

##### nomodule属性

```html
<!-- 
使用type=module，标识main.js是一个模块，这样:
1. 支持原生模块的现代浏览器就能识别 
2. 对于不支持模块的老的浏览器，就会忽略掉这个标签
-->
<script type="module" src="module_a.js"></script>

<!-- 结合使用nomodule属性，提供一个非模块化的代码文件，这样：
1. 支持type=module的现代浏览器，会忽略掉这个script标签。
2. 对于不支持type=module的老的浏览器，就能识别这个标签。
-->
<script nomodule defer src="a.js"></script>
```



因此，结合使用这两种，可以做到既支持老的浏览器，也支持现代浏览器，而且能减少加载的JS代码量，因为不会加载不支持的JS代码。



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

### AMD：异步模块定义

异步模块定义，是一个规范。

#### 定义模块

使用define函数

```javascript
define('SomeMoudle', ['dep1', 'dep2'], function(dep1, dep2){
    
});
```



### UMD：统一的模块

#### 定义模块

一般通过如下的模板来定义模块，原理是通过判断当前所处的环境，决定使用哪种具体的模块化定义方法。

模板也有许多的变体，具体参见[这里](https://github.com/umdjs/umd)

```javascript
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["libName"], factory);
    } else if (typeof module === "object" && module.exports) {
        module.exports = factory(require("libName"));
    } else {
        root.returnExports = factory(root.libName);
    }
}(this, function (b) {
    // 模块的代码写在这里
}))
```

