# TypeScript笔记

### 概述

TypeScript是一门语言，提供了如下的一些特性：

- 完全兼容JavaScript的语法，任何js合法的代码，也是合法的ts代码。
- 提供了一套类型系统，并增加了类型注解语法，可以在函数，变量等定义时，指定他们的类型。
- 提供了一个编译器tsc，编译器能够：
  - 把ts代码编译成JavaScript目标代码，并且能够指定目标代码的ES版本（默认ES3），编译结果中去掉了类型注解信息。
  - 从ts或者js中生成声明文件(.d.ts)
  - 编译过程一些细节可以通过命令行参数控制，或者tsconfig.json配置文件控制。



### 类型系统

TS提供了强大的类型系统，包括：

- 基本类型：string, number, boolean, bigint, symbol
- 数组类型：对数组类型中的元素的类型进行限制。
- 针对索引访问方式```o[1]```、调用访问方式```o(“1”)```、new访问方式```new O()```，可以分别通过```索引签名```、```调用签名```、```构造签名```对参数类型和返回类型进行限制。
- 函数类型：通过函数表达式，调用签名，构造签名对函数进行很好的描述。
- 对象类型：可以通过匿名对象，接口，type 来表示对象类型。
- 类类型：class
- 模块
- 可以对类型进行操作，生成新的类型。包括：泛型，Keyof，Typeof，类型索引访问，条件类型，映射类型，模板字面量类型。
- 字面量类型：是一种常量类型



#### 基本类型

原始类型有：```string, number, boolean, bigint, symbol```。

首字母大写的String, Number, Boolean也合法的，但是他们是JS内置的类名，实现对原始类型的装箱，很少出现在代码中，最好不用做类型注解。

```javascript
let a: string;
let b: String; // 如果合法，但是不要用
```

#### 数组类型

```javascript
// 有如下两种方式
let a: number[];
let s: Array<string>;

a = [1,2,3]
s = ["abc", "def"]

// 对象类型表示数组
let a_by_obj: {[a: number]: number} = [3, 4, 5]
let s_by_obj: {[a: number]: string} = ["3", "4", "5"]
```

#### 对象类型

```typescript
// 可用逗号或者分号作为属性的分隔，也能混合使用，最后一个分割符可写可不写
let person: {"name": string, "age": number} = {name: "Winson", age: 20}
let person1: {"name": string; "age": number,} = {name: "Winson", age: 20}
let person2: {"name": string; "age": number} = {name: "Winson", age: 20}

// 定义好了对象类型后，那么赋值时，对象上的属性不能多，也不能少，如下两个会报错
// let person3: {"name": string, "age": number;} = {name: "Winson", age: 20, "aaa": "222"}
// let person4: {"name": string, "age": number;} = {name: "Winson"}

// 可选属性
let person5: {"name": string, "age"?: number;} = {name: "Winson"}

// 只读属性
let person6: {"name": string, "age"?: number;  readonly sex: string} = {name: "Winson", sex: "male"}
// sex 只读，所以下面语句报错
// person6.sex = "female"
```

#### 联合类型

```javascript
let n_s: number | string;
```

#### 接口类型

使用interface，给类型系统增加新的类型。接口类型有一些特别的用法：

- 用接口表示数组

```typescript
// 和对象类型的定义类似
interface IPerson {
    name: string;
    age: number,
    sex?: boolean;
}

let p: IPerson = {name: "Winson", age: 18}

// 用接口表示数组
interface NumberArray {
    [a: number]: number;
}
let a_num_array: NumberArray = [1, 2, 3]
```

#### 函数类型

```typescript
// mySum是一个函数类型，接受2个number类型参数，返回一个number类型值
let mySum: (a: number, b: number) => number;

mySum = function(a: number, b: number): number {
    return a + b;
}

// 用接口定义函数的形状
interface SumFunc {
    (a: number, b:number): number;
}
let aSum: SumFunc = function(a: number, b: number) {return a + b;}

// 可选参数，必须放在参数列表的最后
let mySumWithOption: (a: number, b: number, c?: string) => string = function (a:number, b: number, c?: string) : string {
    return a + b + c.toUpperCase();
};
// 带默认值的参数，会当做可选参数对待，不过没有位置的限制
let mySumWithDefault: (a: number, b: number, d: string) => string = function (a:number=0, b: number, c: string="aaaa") : string {
    return a + b + c.toUpperCase();
};
// 可选参数，可以理解为默认值是undefined的参数，不能同时指定。

// 剩余参数, 类型可以声明为数组
let mySumWithRest: (a: number, b: number, ...items: number[]) => number = function (a:number=0, b: number, ...items: number[]) : number {
    return a + b + items.reduce((p, c) => c + p)
};

// 函数重载， 前几个是重载的函数签名，最后一个是实现，实现必须兼容重载的函数的签名
// 重载时按先后顺序进行匹配的
function reverse(x: number): number;
function reverse(x: string): string;
function reverse(x: number | string): number | string | void {
    if (typeof x === 'number') {
        return Number(x.toString().split('').reverse().join(''));
    } else if (typeof x === 'string') {
        return x.split('').reverse().join('');
    }
}
```



#### 类型别名

使用type，声明类型别名，给类型系统增加新的类型。一般使用它来给联合类型定义一个别名，方便以后引用。

```typescript
// 定义一个名为ID的新类型，可以为number或者string
type ID = number | string;

let computer1: {serialNo: ID, type: string} = {serialNo: "华为2021", type: "HW9-1"}
let computer2: {serialNo: ID, type: string} = {serialNo: 201301, type: "HW9-2"}

type Point = {
    x: number;
    y: number; // 并且最后一个属性的分隔符，可写可不写。
}
```

#### 字面量类型

字面量类型，表示变量的取值只能为特定的某个或者某些值。用法如下：

```typescript
// 字符串
let a: "Hello" = "Hello"; // a的类型只能为字符串 Hello
a = "Other" // 会报错

// 一个对象类型为某单个字符串很少见，但是为多个字符串中的某一个，却很常见，如：
function printText(s: string, alignment: "left" | "right" | "center") {}

// 多个数字，如下，返回值类型为 0,-1,1中的一个：
function compare(a: string, b: string): -1 | 0 | 1 {
    return a === b ? 0 : a > b ? 1 : -1;
}

// 和联合类型一起使用
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}

// 对于有字面量常量类型参数的函数来说，调用时传参，要特别注意，比如对于上面的printText来说：
const param = {s: "Hello World", alignment: "left"}
printText(param.s, param.alignment); // 虽然param.alignment值为left, 但是它是一个变量，TS会报错。有两种方式解决：
// 1. 传参数时，把参数使用断言，转换成常量，如：
printText(param.s, param.alignment as "left") // 断言成常量了，不报错
// 2. 定义参数时，断言成常量
const param = {s: "Hello World", alignment: "left" as "left"} // 把alignment断言成常量了
const param = {s: "Hello World", alignment: "left"} as const  // 把整个对象都断言成常量了，注意前后两个const，意义是不一样的。
```

#### 类型断言

有时你知道一个对象的具体类型，但是TypeScript不能自动推断，你可以使用类型断言来明确告诉TypeScript。

1. 断言常用在判断一个变量是联合类型中的哪一个具体的类型。

2. 在父类和子类之间使用断言，父类可以断言为子类，子类也可以断言为父类。

3. 任意类型和any之前可以使用断言。

类型断言，其实是欺骗了ts编译器，使用不当时会存在问题，所以必须小心。

```javascript
interface Dog {
    name: string;
    run(): void;
}
interface Fish {
    name: string
    swim(): void;
}
function isFish(animal: Fish|Dog) {
    if (typeof (animal as Fish).swim === 'function') {
        // 上面的判断不能是 typeof animal.swim === 'function'，因为这里只能访问Fish和Dog的公共属性name
        return true;
    }
    return false;
}
let dog = {run: ()=> {console.log("I am running.")}, name: "Haha"}
console.log(isFish(dog))

function swim(animal: Fish|Dog) {
    (animal as Fish).swim(); // 虽然欺骗ts，编译通过了，但是如果传入的是一个Dog，就会报错。所以要小心
}
swim(dog) // 运行时报错


// 父子类之间断言：比如下面的语句，TypeScript只能推断出结果为某种类型的HTMLElement，你使用 as 指定具体为Canvas的Element
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;

// 不过有时，断言的这种直接也太严格，限制了一些很复杂但是合法的断言，如果遇到这种情况，可以先断言成 any 或者 unknown，然后再断言成目标类型，如：
const a = ( express as any) as T;

// 类型断言，只能把类型断言成 更具体 或者 更泛化 的类型，不能在根本不可能的类型直接断言，如：
const x = "hello" as number; // string类型不能断言成number类型
```

#### 类型推断和任意类型

- 任意类型：any，如果没指定类型，而且ts也推断不出类型，ts隐式的推断类型为any，可以通过--noImplicitAny关闭隐式的推断为any。
- ts有隐式的推断类型的能力，大部分情况下，可以利用这个能力，省略类型的注解。

#### 空值类型

对于undefined和null这两种空值类型的处理，TypeScript要根据是否开启了strictNullChecks选项（也就是严格空值检查），来决定具体的类型检查行为。

- 没有开启strictNullChecks

  如果不开启严格空值检查，那么不对null和undefined做任何检查，他们可以赋值给任何类型。这种方式也是最容易出错的地方。所以一般建议是开启严格的空值检查。

- 开启strictNullChecks

  开启了严格空值检查，对于可能为null或者undefined的对象，在使用它的方法或者属性签，必须进行相应的非空判断，否则TS编译出错，这和针对对象的可选属性要进行undefined判断一样。如果你确实断定对象不会为空，那么可以使用非空断言操作符（也就是感叹号），无须非空判断了。

  ```javascript
  function doSomething(x: string|null) {
      if (x === null) {
          // do nothing
      } else {
          console.log(x.toUpperCase())
      }
  }
  // 确定参数x不会为空，那么使用非空断言，就无须空值判断了
  function doSomething(x: string|null) {
      console.log(x!.toUpperCase()) // x后面带感叹号
  }
  ```

#### 枚举类型Enums

是TS语言层面的，除非你很了解他的用法，否则尽量不用。更多参考看[这里](https://www.typescriptlang.org/docs/handbook/enums.html)





### 类型系统进价

#### 函数类型

##### 函数类型表达式

```typescript
// 匿名方式
let f: (a: string) => string;
function greeter(fn: (user: string)=> string) {}
// 通过type给表达式取个名字
type GreetFunction = (user: string) => string;
function greeter(fn: GreetFunction, user: string) {
    fn(user)
}
// 调用
greeter((user:string) => {
    console.log(`hello ${user}`);
    return `hello ${user}`;
}, "Winson")
```

##### 调用签名Call Signature

一个函数除了可以调用外，还能包含属性，但是函数类型表达式的语法是不允许声明属性的。为了解决这个问题，我们可以在一个对象类型里声明一个调用签名，如：

```typescript
type DesribableFunction = {
    description: string,
    (s: string): string // 这是一个调用签名，调用签名和函数类型表达式不同之处是使用 ‘：’ 代替了 ‘=>’ 来指定返回值类型
}
function doSomething(fn: DesribableFunction) {
    console.log(fn.description + " returned " + fn("Winson"))
}
```

##### 构造签名

一个函数，可以通过new操作符进行调用，生成一个新的对象。可以通过在调用签名前加上new 来描述这种情况。

```typescript
type SomeConstructor = {
    new (s: string): SomeObject; // 返回肯定是一个对象
}
function fn(ctor: SomeConstructor) {
    return new ctor("Hello");
}
```

有些函数，既能直接调用，也能使用new，比如js内置的Date就是。那么就可以同时使用调用签名和构造签名，如下。

```typescript
interface CallOrConstruct {
    new (s: string): Date;
    (n?: number): number;
}
```



#### 类类型

ts完全支持ES6的类，并且给它增加了类型注解和其它的一些语法，让你能够表达类和其它类型之间的关系。需要了解的内容包括：

- 类的成员：字段，只读字段，字段的初始化，构造器，方法，getter/setter，索引签名
- 构造器重载，父类调用super()
- 类的继承：implements（实现接口），extends（类继承），方法重载，类的初始化顺序，继承内置的类。
- 成员的可见性：public, protected, private。
- 静态成员及其可见性
- 特殊的静态成员：name, call, length
- 静态块
- 泛型类
- 类中的this
- 类中箭头函数
- this参数
- 方法参数属性：readonly, public, private, protected.
- 类表达式
- 抽象类和成员
- 抽象构造器签名
- 类之间的关系

```typescript
class AClass {
    x:number; // number
    y = 0; // 自动推断为number类型
    z, // any
}
```

#### 类型操作

可以对类型进行操作，生成新的类型。操作类型的方式有如下几种：

- 泛型
- keyof 操作符
- typeof 操作符
- 对类型进行索引访问
- 条件类型
- 映射类型
- 模板字面量类型

除此之外，TS还提供了许多内置的类型转换工具，包括：Partial， Required，Readonly，Record，Pick，Omit，Exclude，Extract，NonNullable，Parameters，ConstructorParameters，ReturnType，InstanceType，ThisParameterType，OmitThisParameter，ThisType。

内置的对字符串类型的工具：Lowercase，Uppercase，Capitalize，Uncapitalize





### 模块

在TS中，和ES6一样，任何顶级包含 import 和 export 语句的文件，都是模块。相反，如果不包含这两个语句的，就是普通脚本，而不是模块。模块有自己的作用域，非模块在全局作用域中执行。

如果你有一个普通脚本，但是希望被当做模块对待，那么在文件中加上如下一行代码即可：

```javascript
export {} // 让非模块变成模块，并且不导出任何东西。
```

使用TS编写模块化代码，有三个重要的事情需要考虑：

- 语法：使用哪种模块定义语法
- 模块解析：模块名和磁盘上的文件是什么对应关系。
- 模块输出目标：我们输出的模块看起来是什么样的？

#### 语法

##### ES 6 语法

ES6 的模块定义和使用语法都可以使用

##### TS特定的语法

- 导出类型和接口，export type, export interface等
- import type
- inline type imports
- ES的语法结合CommonJS的行为：import fs = require("fs")
- CommonJS的语法也可以



### 声明文件: xxx.d.ts

#### 理论

声明文件描述了一个模块的精确的API Shape，比如暴露了哪些类型（Type），哪些值（Value），哪些命名空间（namespace）。要完全理解如何定义声明文件，我们必须理解TS中的这三个核心的概念。

##### Types：类型

TS中的类型通过如下的声明引入的：

- 类型别名声明：```type sn = number | string```
- 接口声明：```interface I {x: number[];}```
- 类声明：```class C {}```
- 枚举声明：```enum E {A, B, C}```
- 一个指向类型的导入声明（import）

以上每一个形式的声明，都会创建一个新的类型名称。

##### Values：值

TS中的值，通过如下方式创建：

- let, const, var 声明
- 里面包含了值的 namespace, module 声明
- enum 声明
- class 声明
- 一个指向值的导入声明（import）
- function 声明

##### Namespaces: 命名空间

类型可以包含在命名空间中，比如这个声明：```let x: A.B.C```，我们说类型```C```来自于命名空间```A.B```。

这种区分很微妙也很重要，这里的A.B不一定是一个类型或者值。

#### 简单的合并：一个名字，多个含义

对于一个名字A，我们发现可能有三种含义：类型，值，命名空间，具体这个名字如何解析要依赖于使用它的上下文。比如：```let m: A.A = A;```，A首先是一个命名空间，然后是一个类型，最后是一个值。

这看起来比较迷惑，但只要不过度重载，实际上是很方便的。

##### 内置的合并

比如 class C {}，C既是类型，表示类，也是值，表示类的构造函数。当然 enum 也有类似的行为。

##### 用户的合并

```javascript
// foo.d.ts，Bar既是类型，也是值
export var Bar: { a: Bar };
export interface Bar {
  count: number;
}

// 导入时，就很方便了
immport { Bar } from './foo'
let x: Bar = Bar.a; // 前一个Bar是类型，后一个是值
console.log(x.count);
```



#### 高级的合并

某些类型的多个声明能够合并。比如：class C {} 和 interface C {} 能够共存，共同给类型 C 提供自己的属性。这样是合法的，只要他们之间不会产生冲突就行。一个通用的规则是：

- 两个同名的值，会产生冲突，除非他们声明为命名空间。
- 同名的类型直接如果通过类型别名声明，会产生冲突。
- 命名空间任何时候不会冲突。

##### 通过接口或类添加

给接口或者类 添加 新的成员

```typescript
interface Foo {
  x: number;
}
// 在别处定义
interface Foo {
  y: number;
}
class Foo {
  z: number
}
let a: Foo = ...;
console.log(a.x + a.y + a.z); // 会合并，存在 x,y,z 属性
```

##### 通过命名空间添加

通过命名空间可以添加新的类型，值，或者命名空间，并且不会产生冲突。比如，给类增加一个静态成员：

```typescript
class C {}

// 其它某个地方
namespace C {
    export let x: number;
    export interface D {}
}
let y = C.x; // OK，C作为名称空间，存在x成员
let z: C.D; // 也OK
```

##### 例子解析

```typescript
namespace X {
  export interface Y {}
  export class Z {}
}
/*
对于上面的声明，我们定义了下面这些：
1. 一个值X：因为namespace里包含了一个值，即Z构造函数，所有X本身也是一个值。
2. 一个命名空间X：因为命名空间中包含了一个类型 Y
3. 一个类型Y：在命名空间X中
4. 一个类型Z：在命名空间X中
5. 一个值Z：是 X值 的属性，也就是Z的构造函数。

总结来说，X既是值，也是命名空间，Y是类型，Z既是值也是类型。
/*
```

