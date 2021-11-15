# CommonJS module



- 它是Node.js的模块化系统，在这个系统里，每一个js文件都是一个独立的模块。

- 在js文件中，通过一个名为exports对象，暴露本模块对外提供的函数或者对象。
- js文件中的其它变量对于模块来说都是私有的，外部不能访问。因为在底层的模块实现上，Node.js会把模块包装在一个函数里。
- 可以给exports对象添加新的属性，如exports.aaaa = Function or Object，或者直接给module.exports赋值，如module.exports = class A。
- 模块系统是通过require('module')实现的。



### 访问主模块

可以使用require.main === module来判断当前模块是否是主模块，module是当前模块，require.main是主模块。这个和python中判断主模块的方式类似：```if __name__ == "__main__"```，python中主模块的名字为```__main__```，而当前模块的名字可以通过```__name__```变量获取。



module上有一个filename的属性，可以获取模块对于的文件名。



### require的逻辑

我们一般通过require来加载依赖，我们可以通过require.resolve函数获取要加载的依赖的精确文件名。

依赖可以是一下三种，使用不同的逻辑进行加载：

- *.js：JS文件，按照JavaScript解析
- *.json：json文件，使用JSON.parse解析成一个JS 对象
- *.node：二进制插件，相当于动态链接库，以node为后缀名。（相对于windows的dll，linux的so），使用process.dlopen()加载。



Node.js通过require的定位模块路径的几种形式：

- require("fs")  加载 node内核模块，这些模块编译到了Node.js执行程序里
- require("./foo.js"), require("../foo.js") 按相对路径进行加载
- require("/home/winson/foo.js")  按绝对路径进行加载
- require("rollup")  加载第三方模块，必须存在于一个node_modules文件夹中
- 最终如果模块找不到，加载失败，抛出Error异常对象，code属性为: 'MODULE_NOT_FOUND'



另外由于历史原因，Node.js还会从如下目录中搜索模块：

- 从NODE_PATH环境变量制定的目录里，搜索模块。这是在使用node_modules文件夹组织模块之前，Node.js定位模块的逻辑，现在还能使用，但是不推荐了。
- 从全局目录（GLOBAL_FOLDERS）里搜索，这些目录有：$HOME/.node_modules, $HOME/.node_libraries, $PREFIX/lib/node。



那么require是如何定位要加载的依赖所在的路径呢？下面通过一些例子来说明。

```javascript
// 比如下面的代码，require会导入具体哪个文件呢？
// 我们可以通过require.resolve('./foo')获取到
const foo = require('./foo')
```

下面举一些常用的例子说明加载的路径：

```javascript
// 一、加载核心模块，
require("fs") 

// 二、按相对路径（当前目录，上级目录）加载：
// 1.带了js或者json或node的扩展名，直接按具体的方式加载
require("./foo.json") // 按json加载，返回一个JSON对象

// 2.没有扩展名，在当前路径下，依次判断以js, json, node为扩展名的文件是否存在，存在就按相应的方式加载
require("./foo") // 根据情况，实际加载的可能是foo.js, foo.json, foo.node

// 3.如果不存在上述三个扩展名对应的文件，那就判断把foo当做文件夹进行处理
// 此时又分为几种情况：
// 3.1 foo文件夹下存在package.json, 判断main字段对应的属性
	require("./foo") // 3.1.1 main有值，带后缀，那么实际加载的就main属性对应的文件，如：./foo/a.js
	require("./foo") // 3.1.2 main有值，无后缀，依次判断js, json, node是否存在，加载具体的文件，如：./foo/b.node
	require("./foo") // 3.1.3 main是falsy的值，依次判断index.js, index.json, index.node是否存在，如：./foo/index.json
// 3.2 foo文件夹下不存在package.json
	require("./foo") // 按3.1.3处理，可能加载：./foo/index.js


// 三、加载第三方依赖
// 首先沿着当前路径往上直到根目录，获取到所有的node_modules文件夹。然后遍历所有的文件夹，判断第三方模块是否存在。
// 判断模块是否存在时，也要根据.js, .json, .node来逐一检查，如果不存在文件，还要根据是否存在目录，目录中是否存在package.json
// 找到最终的模块路径。加入在/home/winson/proj1/index.js文件中有如下语句：
require('bar.js')
// 那么所有的node_modules路径有:
// /home/winson/proj1/node_modules
// /home/winson/node_modules
// /home/node_modules
// /node_modules
```



### 模块缓存

- 模块是有缓存的，多次require同一个路径的文件，返回的是同一个对象。
- 这个对象可以是 ”部分完成“，这允许模块直接相互依赖，甚至构成循环依赖。
- 缓存的模块对象保存在require.cache中，只要不修改require.cache，模块里的代码不会多次加载执行。如果需要多次执行，导出一个函数，执行函数多次即可。
- 【警告】模块缓存是以文件的绝对路为key存储的。如果require("./foo")在不同的文件中，解析出来的绝对路径不同，那么他们肯定是不同的模块。另外在文件路径大小写不敏感的文件系统中，require("./foo")和require("./FOO")，解析出来的绝对路径虽然指向同一个文件，但是他们也会被当成不通过的模块，因为缓存的key是区分大小写的，他们是不一样的。



### 模块包装器

在模块的代码执行前，Node.js会把模块的代码包裹进一个函数包装器里，就像这样的：

```javascript
(function(exports, require, module, __filename, __dirname) {
    // 这里是模块的代码，它形成了一个模块的作用域，传入了5个参数，代码可以直接使用
})
```

这样包装的目的是：

- 使得模块文件的定义的顶级变量（let, var, const定义的变量）的作用域只是针对这个模块，而不会影响到global对象。
- 模块里的代码能够很方便的直接访问包装函数的参数：如：```exports```,``` require```, ```module```, ```__filename```, ```__dirname```



### 模块作用域

#### ```___filename```

当前模块的文件名



#### ```__dirname```

当前模块文件对应的路径



#### exports

是module.exports的引用，方便使用，编码时减少输入。不过要注意何时可以使用exports, 何时必须使用module.exports



#### module

一个module对象。特别是module.exports定义模块导出的什么内容，只有导出的内容能够使用require导入。

##### module.children

模块的子模块，也就是它们第一次被加载时，是通过当前模块加载的。

##### module.exports

是模块系统创建的一个对象。有时希望导出的就是某个类的实例，那么可以把实例直接赋值给module.exports。给module.exports赋值必须马上进行，不能在回调中执行。比如：a.js

```javascript
const EventEmitter = require('events');
// 直接把类的实例赋值给module.exports
module.exports = new EventEmitter();

// Do some work, and after some time emit
// the 'ready' event from the module itself.
setTimeout(() => {
  module.exports.emit('ready');
}, 1000);
```





#### require(id)

用来导入module，JSON或者本地文件。



##### require.cache

是模块缓存的地方，缓存可以删除，也可以替换。

##### require.main

保存了主模块

##### require.resolve(id, options)

返回解析出的模块的路径，和require(id)的逻辑相同，只不过不进行加载。不存在，抛异常。

##### require.resolve.paths(id)

返回解析模块时，搜索过的所有路径的数组。如果id是一个核心模块，返回null。









