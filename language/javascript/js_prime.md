# JavaScript基础

### 严格模式和非严格模式

#### 非严格模式

在某些时候，你也会见到把非严格模式称为sloppy mode，这是一种非官方的叫法。它表示不是strict mode下的模式。



#### 严格模式

##### 简介

- 严格模式，是在ES 5中官方引入的一种模式。

- 是JavaScript的一种受限的变体。严格模式不仅仅是一个子集：它有意地具有与普通代码不同的语义。
- 支持严格模式的浏览器和不支持严格模式的浏览器在运行严格模式代码时会有不同的行为，所以，你不应该不经过特性测试，直接使用依赖于严格模式的相关语义特性。
- 严格模式和非严格模式代码可以共存，这就允许我们增量地把非严格模式代码改成严格模式代码。

##### 语义变化

严格模式对普通的JavaScript语义做的变化有：

- 把一些JavaScript的静默错误，改为抛出错误。也就是把原来执行失败却没有报错的问题，变成抛出错误。
- 修复一些导致JS引擎很难进行优化的错误：相同代码在严格模式下有时会比在非严格模式下运行的更快。
- 禁止提前使用可能在将来的ES版本中才能出现的一些语法。

##### 开启严格模式

严格模式可以应用于整个JS脚本文件，也可以应用于单个函数。不能应用于块语句（在块语句中使用，没有任何作用）。开启方式为：```'use strict';```

1. 按整个脚本开启严格模式

也就是在脚本文件最前面使用```'use strict';```，这种方式在合并脚本的时候，会发生冲突。最好在函数作用域开启严格模式。

2. 按函数开启严格模式

   ```javascript
   function strict() {'use strict'; ...}
   function noStrict() {...}
   ```

3. 按模块开启严格模式

   ES 6引入了模块技术，模块中的函数，默认就是严格模式。

   ```javascript
   function strict() {
       // 因为这是一个模块，所以函数里默认就是演示模式
   }
   export default strict;
   ```

4. ES 6中的类的所有部分都是严格模式



#### 严格模式下的改变举例

严格模型既有语法的改变，也有运行时行为的改变。

##### 把静默错误改为抛出异常

把在非严格模式下的一些mistakes变为error， 也就是一些静默错误会变成抛出异常， 如：

```javascript
'use strict'; // 开启严格模式

//1.给未声明的变量赋值, 会抛出异常
mistypeVariable = 17;  

//2.给非可写的全局变量赋值, 如: undefined, NaN, Infinity等等
var undefined = 5; // 抛出TypeError

//3.给非可写的对象属性赋值
var obj1 = {}
Object.defineProperty(obj1, 'x', {value: 42, writable: false}) // x不可写
obj1.x = 9; // 抛出TypeError

//4.给getter-only属性赋值
var obj2 = {get x() {return 17;}}; // obj2的x只有getter属性
obj2.x = 5; // 抛出TypeError

//5.给不扩展的对象增加新的属性
var fixed = {}
Object.preventExtensions(fixed); // fixed不可扩展
fixed.newProp = 'ohai'; // 抛出TypeError

//6.删除不能删除的属性
delete Object.prototype;  // 抛出TypeError

//7.函数的参数名重复
function sum(a, a, c) { // 语法错误，a重复了
    'use strict';
    return a + a + c;
}

//8.使用前导0标识8进制数字
var sum = 015 + 197; // 语法错误
var sumWithOctal = 0o10 + 8; // 结果为16，严格模式下，必须使用前导0o标识8进制数字

//9.给原始数据类型增加新的属性
false.true = ''; // TypeError
(14).sailing = 'home'; // TypeError
"with".you = "far away"; // TypeError

//10.属性名重复，在ES 5的严格模式下，会抛出语法异常，但是在ES 6下不会，因为ES 6引入了可计算的属性名。
var o = {p: 1, p: 2}; // ES 5的严格模式下，会抛语法异常。
```

##### 简化变量的使用

简化了变量的使用，也就是简化了从变量名映射到特定的变量定义位置的过程，这对于JS引擎优化代码很有帮助。具体来说，有如下几个方面：

```javascript
'use strict'; // 开启严格模式

//1. 禁止使用with, 因为with块中一个变量，可能来自传入的对象，或者外部甚至是全局的。
//增加了解析变量的复杂性，也不方便优化。
var x = 17;
with (obj) { // 语法错误
    x;
}

//2.eval在严格模式下，不会给外部作用域引入新的变量
var x = 17;
//此例子是在eval函数里显示开启了严格模式，
//如果外部已经是严格模式了，那么里面的use strict可以省略。
var evalX = eval("'use strict'; var x=42; x;"); 
console.assert(x === 17); // x还是等于17
console.assert(evalX === 42); // evalX才等于42

//3.禁止使用delete
var x;
delete x; // 语法错误
eval('var y; delete y;'); // 语法错误
```

##### 简化eval和arguments

在普通情况下，他们两个会涉及大量的神奇的行为，比如，eval可以增加和移除绑定，改变绑定的值等等。严格模式使得这两者不再那么具有神奇的魔力了。比如：

```javascript
'use strict';

//1.不能给这两个名字重新赋值，否则会抛出语法异常。 如下都会抛出语法异常：
eval = 17;
arguments++;
++eval;
function arguments(){}
function x(eval) {}
var f = new Function('arguments', "'use strict'; return 17;")

//2.arguments不会和函数的显式参数进行绑定了
function f(a) {
    'use strict';
    a = 42; // 如果是非严格模式，修改a，也就是修改了arguments[0], 严格模式下不会
    return [a, arguments[0]] // 
}
var pair = f(17);
console.assert(pair[0] === 42)
console.assert(pair[1] === 17) // 如果是非严格模式，pair[1] === 42

//3.arguments.callee不再受到支持。
var f = function() {return arguments.callee;}
f(); // TypeError
```

##### 使得JavaScript更加安全

严格模式使得编写安全的JS代码变得更加容易。

###### this指向的改变

在非严格模式下，函数中的this绑定的必须是一个对象，如果是原始类型，就会发生装箱。如果没有显式绑定，这个对象就是全局对象。这样就会导致装箱的性能问题和暴露全局对象的安全问题。



所以在严格模式下，this不会装箱，也不会绑定到全局对象上。

```javascript
'use strict';
function fun() {return this;}

console.assert(fun() === undefined); // 没有显式绑定，this就是undefined, 在非严格模式下为全局对象
console.assert(fun.call(2) === 2); // 绑定原始类型，this也不会装箱为一个对象
console.assert(fun.apply(null) === null); 
console.assert(fun.bind(true) === true); 
```

###### caller和arguments不能通过函数访问

在非严格模式下，一个扩展库，可以通过fun.caller和fun.arguments获取fun的调用者和调用传参。这就可能导致安全问题。



所以在严格模式下，对caller和arguments不能被删除，也不能被访问。否则会抛出异常。

```javascript
function restricted() {
  'use strict';
  restricted.caller;    // throws a TypeError
  restricted.arguments; // throws a TypeError
}
function privilegedInvoker() {
  return restricted();
}
privilegedInvoker();
```

###### arguments关键字不能使用caller

在一些很老的ES实现中，arguments.caller表示的是一个对象，它的属性是函数中定义的变量，这也会导致安全问题。



所以在严格模式下，arguments关键字上，不能使用caller属性。如：

```javascript
'use strict';
function fun(a, b) {
  'use strict';
  var v = 12;
  return arguments.caller; // throws a TypeError
}
fun(1, 2); // 不会暴露fun中的变量a, b, v等
```

##### 为ES的将来版本铺路

ES将来的版本可能会引入新的语法。严格模式下的一些限制，就会更容易地把代码适配到新的版本中。比如：

###### 保留关键字

在严格模式下，一些将来可能使用的关键字，不能作为变量的标识符。如：implements, interface, public, static, yield, let, private, package, protected等。	

###### 函数必须定义在脚本或者函数中的顶级

严格模式下，不能在if语句块，for语句块等非顶级作用域中定义新的函数。









