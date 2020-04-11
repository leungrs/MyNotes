### Java类和对象
类是创建对象的蓝图(blueprint)，对象是类的实例。

#### 类的声明
一个类的声明，由以下组件组成
1. 修饰符：public private 等等，不包括protected，因为protect只修饰方法。
2. 类名，如Person
3. 父类名，前面接extends，一个类只能有一个父类
4. 实现的接口列表，从逗号分隔。可以实现多个接口。
5. 类体，从大括号包起来，按顺序包含field，constructor，method等等。

#### 类的构造器
1. 如果一个类没有定义构造器，那么编译器会提供一个默认的无参构造器，在这个无参构造器中会调用它的父类的无参构造器。所以这种情况下，必须确保它的父类提供了一个无参构造器。Object类是有一个无参构造器的。
2. 如果一个类定义了无参或者有参构造器，那么编译器就不会提供默认的无参构造器了。
3. 如果一个类的构造器是private的，那么不能通过它来new出新对象。


#### 类的方法
由六部分组成：
1. 修饰符，如private public 等等。
2. 返回值类型
3. 方法名
4. 参数列表
5. 抛出异常列表
6. 函数体

##### 类的方法参数
1. 参数类型可以是任意8种原始数据类型和任意的引用类型（字符串，对象，数组）
2. 如果往一个方法参数里传递一个方法，可以通过lambda表达式或者method reference(方法引用)。
3. 使用三个点来声明任意数量参数:`void prinf(String fmt, Object... objs)`
   - 该参数必须放置在参数列表最后
   - 可以单个的传多个，也可以传一个数组进来，因为objs本身就是当做一个数组类对待的。
4. 参数名：如果和field同名，称为参数隐藏了field。这样的代码可读性不好。这种情况下，field可以通过this来访问。
5. 原始数据类型按值传递（字符串是不可变对象，也是按值传递），其他的引用对象按引用传递（数组，对象，不包括字符串）

#### 类的访问控制
访问级别修饰符决定了别的类是否能够访问类的成员和方法。有两种类型的修饰符：
1.类上的修饰符，有两种级别：public和package-private(也就是没有修饰符)
2.member上的修饰符，有四种级别：public, protected, package-private, private

- public：表示别的类可以访问
- protected：表示同一个包里的类可以访问，或者不在一个包，但是是它的子类也可以访问
- package-private: 表示只能在同一个包里类才能访问
- private：表示别的类不能访问，只能本类可以访问。

#### 类的静态成员（也叫类成员，使用static修饰）
类成员（包括属性和方法）属于类，内存中只存在一份，不属于类的实例。但是可以通过实例对象来方法，不过不推荐，
推荐的做法是通过类名来直接访问。

静态方法可以访问静态属性，但是不能访问实例属性。

static经常和final一起使用来定义常量。表示这个属性的值不能被更改。
final也可以修饰方法，表示这个方法不可变，也就是说子类不能重写（override）。

#### 类成员的初始化
1.静态成员的初始化：初始化逻辑复杂的可以使用静态初始化块。简单的可以声明的时候马上进行初始化。
2.实例成员的初始化：初始化逻辑复杂的使用构造器来进行初始化。另外还可以使用实例初始化块，也就是去掉了静态初始化块的static。编译器会把实例初始化块拷贝到每一个类构造器中。简单的可以声明的时候马上进行初始化。


#### Java对象
对象通过new和构建起来创建和初始化。创建出来的对象通过点运算符获取方法和field。

##### 对象的垃圾回收
当对象没有引用存在时，会被垃圾回收。引用是保持在变量里，变量的超出了作用域范围，变量所代表的引用也就不存在了。
我们也可以通过设置变量为null来释放一个引用。

Java运行环境有一个垃圾收集器，定期的检查对象的引用，对于没有引用的对象，就释放其占用的内存。


### 嵌套类（Nested Class）
嵌套类，也就是定义在一个类（称为Outer Class外部类）中的类。
作为外部类一个成员，它的访问修饰符可以和外部类中的其他成员一样，
有private,protected,package-private,public四种。记住，外部类访问修饰符只有public和package-private两种。

嵌套类又细分为两类：
1.使用static修饰的，叫静态嵌套类（static nested class）。
2.没有使用static修饰的，叫内部类（Inner class）。

静态嵌套类，不能访问外部类的成员。内部类可以访问外部类的成员，包括外部类的私有成员。

#### 为什么要使用嵌套类
1.只被一个其他的类使用的类，可以和使用它的类放在一起，作为它的嵌套类。可以减少类文件的数量，保持包的干净。
2.提高了封装性。比如有类A和类B，类B需要访问类A的成员c，但是c应该是private类型的，
  如果不用嵌套类的话，那么c必须声明为public，这样就降低了封装行。另外，类B也被封装在了A里面。
3.能够提供代码可读性和可维护性，因为把关联的类放在了一起。

#### 静态嵌套类
静态嵌套类其实行为上和一个正常的类是一样的，只不过使用的时候，需要使用外部类来访问，
并且可以使用private，和protected来限制他的被访问级别。

使用的例子：
```java
OuterClass.StaticNestedClass obj = new OuterClass.StaticNestedClass();
```

#### 内部类（非静态嵌套类）
和实例变量与方法一样，内部类时和一个实例关联在一起的。能够直接访问关联的实例的属性和方法。
由于是和实例关联，所有它本身不能定义任何的静态属性和方法。

内部类的实例只能存在于外部类的实例当中，并且能够访问外部类实例的属性和方法。

要实例化一个内部类的对象，必须首先实例化一个外部类对象。然后这个外部类对象.new 来进行实例化，如下：
```java
OuterClass outObj = new OutClass();
OutClass.InnerClass innerObj = outObj.new InnerClass();
```

名字隐藏，小作用域的变量名字会隐藏包含它的外部的大的作用域里的变量名字，
也就是不能直接使用相同的名字来访问外部作用域里的变量。如果在小作用域里需要访问大作用域里的名字，
必须使用一定的前缀。比如下面例子中的x：

```java
public class ShadowTest {
    public int x = 0;
    class FirstLevel {
        public int x = 1;
        void methodInFirstLevel(int x) {
            System.out.println("x = " + x);
            System.out.println("this.x = " + this.x);
            System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
        }
    }
    public static void main(String... args) {
        ShadowTest st = new ShadowTest();
        ShadowTest.FirstLevel fl = st.new FirstLevel();
        fl.methodInFirstLevel(23);
    }
}
```

#### 内部类的序列化
强烈不建议序列化内部类（包括匿名类和局部类）。因为这可能导致不兼容问题，也就是一个编译器编译出来的内部类，
在另一个编译器中不被识别。

#### 特殊的内部类
内部类除了可以直接定义在一个类中以外，还可以定义在类的方法里。  
如果定义在类的方法里，称其为局部类（或者本地类 local class)。
如果定义在类的方法里，并且没有名字，那么就称其为匿名类（anonymous class）

#### 局部类
局部类可以和局部变量进行类比，局部类可以放置在方法中的任何代码块（大括号包起来的）中，比如：if，while，for 等，
并且作用域只在当前代码块中生效。

1. 局部类可以方法外部类中的成员，静态方法中的局部类访问外部类的静态成员，实例方法中的局部类，可以方法静态成员和实例成员。
2. 局部类访问方法里的局部变量（包括方法的参数），那么该局部变量在局部类中不能被修改。一般可以声明为final类型防止被修改。
  当然如果不声明为final类型，在JDK8及以后，只要在使用的过程中注意不要改变局部变量的值也行，
  效果上等同于局部变量被声明成了final类型。但是在JDK8以前，还是必须声明为final类型。
3. 局部类本身不能是静态的，因为可以访问外部类的实例变量。
4. 不能像定义局部类一样在方法中定义局部接口，因为接口天生都是静态的。
5. 通常局部类中不能定义静态成员，但是对于原始数据类型和字符串，可以定义为静态常量(static final)，并且使用内联初始化。
也就是: `private static final int a = 10;`

#### 匿名类
没有名字的局部类，称为匿名类。使用匿名类可以让代码更简洁。如果局部类只使用一次，可以使用匿名类。

局部类是类的声明，匿名类其实是一个表达式。我们在另一个表达式中声明一个匿名类。
匿名类这个表达式，它的语法有点像是调用一个构造器，但是在接下来的代码块中包含了一个类的定义。

匿名类的语法包含：
1. new 操作符。
2. 实现的接口的名字，或者需要继承的类的名字。
3. 大括号里面包含一个构造器的参数列表，就像一个正常类的实例创建表达式。当然如果是实现一个接口，那么括号中就没有参数。
4. 类的定义体。

举例如下
```java
public class OutClass {
  // 内部类，是一个接口
  public interface HelloWorld {
    String sayHello(String name);
  }

  public void sayHello(String name) {
    // 这是一个局部类，因为有名字
    class ChineseSayHello implements HelloWorld {
      public String sayHello(String name) {
        return "你好，" + name;
      }
    }
    HelloWorld hello = new ChineseSayHello();
    // 这是一个匿名类：new + interface + () + 类体
    HelloWorld hello1 = new HelloWorld() {
      public String sayHello(String name) {
        return "Hello, " + name;
      }
    };
    System.out.println(hello.sayHello(name));
    System.out.printlin(hello1.sayHello(name));
  }
  public static void main(String[] args) {
    new OutClass().sayHello("Winson");
  }
}
```

匿名类的使用场景：

- 在GUI应用程序中用的比较多。
- 匿名类可以使用lambda表达式来替代。

### lambda表达式

#### lambda表达式的语法
一个lambda表达式由以下几个部分组成：
1. 小括号括起来的，用逗号隔开的形式参数列表。参数的数据类型可以省略，如果只有一个参数，小括号也可省略。
2. 箭头符号：->
3. 一个body，由单一的表达式或者语句块组成。

举例：
```java
p -> {
  return p.getAge();
}

p -> p.getAge()

p -> System.out.println(p.getName())
```

lambda表达式看起来很像一个方法的声明，只不过没有方法名。你可以把lambda表达式认为是一个匿名的方法。

lambda表达式

#### lambda表达式的理想使用场景
比如有如下的一个人员类：
```java
public class Person {
  public enum Sex {
    MALE, FEMALE
  }

  String name;
  Sex gender;
  int age;
  String emailAddress;

  public void printPerson() {
    System.out.println(name);
  }
  public int getAge() {
    return this.age;
  }
  public Sex getGender() {
    return this.gender;
  }
  public String getEmailAddress() {
    return this.emailAddress;
  }
}
```

现在有个人员的列表List<Person>，你需要从中选择满足一定条件的人员，执行任意的操作。

1. 方式一：为每个特定的条件创建一个方法
```java
// 条件在代码里写死了，如果需要打印年龄在某个范围的人员，就不能满足了
public static void printPersonOldThan(List<Person> list, int age) {
  for (Person p: list) {
    if (p.getAge() > age) {
      p.printPerson();
    }
  }
}
```

2. 方式二：让条件变得更通用一点，指定年龄的上下限
```java
// 指定了年龄范围，但是如果还需要指定特定的性别，又得增加其他的方法
// 这种为每个特定条件，增加一个方法的方式，代码就不简洁了。
// 可以考虑把定义搜索条件的工作放到另外一个类中
public static void printPersonWithinAgeRange(List<Person> list, int low, int high) {
  for (Person p: list) {
    if (low < p.getAge() && high > p.getAge()) {
      p.printPerson();
    }
  }
}
```

3. 方式三：把指定搜索条件的代码放到另一个类中
```java
// 这种方式，你需要一个接口，然后为每个条件添加一个实现类，还是比较繁琐。
// 为了减少类的数量，可以使用匿名类
public interface CheckPerson {
  boolean test(Person p);
}
public class Seletor implements CheckPerson {
  public boolean test(Person p) {
    return p.getGender() == Person.FEMALE && p.getAge() > 23;
  }
}
public static void printPersons(List<Person> list, CheckPerson tester) {
  for (Person p: list) {
    if (tester.test(p)) {
      p.printPerson();
    }
  }
}
printPersons(list, new Selector());
```

4. 把指定搜索条件的代码放到匿名类中
```java
// 这种方式的代码就比较简洁了，考虑到接口中只有一个方法，
// 此时还可以使用lambda表达式进一步简化。
printPersons(list, new CheckPerson() {
  public boolean test(Person p) {
    return p.getGender() == Person.MALE && 23 < p.getAge() && 30 > p.getAge();
  }
})
```

5. 在lambda表达式中指定搜索条件
```java
// 使用标准的函数式接口(Standard Functional Interface), 连CheckPerson接口也可以不需要
printPersons(list, p -> p.getAge() > 23 && p.getGender() == Person.FEMALE);
```

6. 使用标准的函数式接口与lambda表达式
JDK自带了几个常用的函数式接口，在java.util.function接口。CheckPerson可以使用`Predict<T>`接口代替。
```java
// 使用Predict<Person>接口代替了CheckPerson接口，效果一样
public static void printPersonWithPredict(List<Person> list, Predict<Person> tester) {
  for (Person p: list) {
    if (tester.test(p)) {
      p.printPerson();
    }
  }
}
printPersonWithPredict(list, p -> p.getGender() == Person.MALE);
```

从上面1到6个步骤，我们一步步的引入了lambda表达式，演示了使用它的优势。

现在继续分析printPersonWithPredict方法，我们来看看怎么使用lambda表达式进一步优化代码。
上面的方法中使用了函数式接口`Predict<T>`后，我们可以方便的传入不同的搜索条件，然后执行打印的操作。
但是如果要执行不同的操作，可不可以也使用lambda表达式呢？

答案是可以的，使用JDK中自带的函数式接口`Consumer<T>`，重新定义函数：
```java
public static void printPersonWithPredict(List<Person> list, Predict<Person> tester, Consumer<Person> consumer) {
  for (Person p: list) {
    if (tester.test(p)) {
      consumer.accept(p);
    }
  }
}
printPersonWithPredict(list, p -> p.getAge() == 23, p -> p.printPerson() )
```

使用JDK中的Function<T, R>，我们可以更加丰富上面的函数的功能：
```java
public static void processPersonWithFunction(List<Person> list, Predict<Person> tester,
  Funtion<Person, String> mapper, Consumer<String> consumer) {
  for (Person p: list) {
    if (tester.test(p)) {
      String data = mapper.apply(p);
      consumer.accept(data);
    }
  }
}

// 获取人员的邮箱地址，进行打印
processPersonWithFunction(list, 
  p->p.getAge() == 23,
  p->p.getEmailAddress(),
  email->System.out.println(email)
);
```

使用泛型进一步改造函数，让它能够处理任何类型的列表：
```java
public static <X, Y> void processElements(Iterable<X> source, Predict<X> tester,
  Funtion<X, Y> mapper, Consumer<Y> consumer) {
  for (X p: source) {
    if (tester.test(p)) {
      Y data = mapper.apply(p);
      consumer.accept(data);
    }
  }
}

processElements(list, 
  p->p.getAge() == 23,
  p->p.getEmailAddress(),
  email->System.out.println(email)
);
```

8. lambda表达式作为聚合操作的参数

上面处理人员类别的函数，使用聚合操作实现如下：
```java
// filter, map, forEach等都是聚合操作, 聚合操作处理的元素必须是从一个stream中来的。
// stream()函数，就是用来生成一个stream流。
// stream它不是一个用来存储元素的数据结构，它通过一个管道来承载一个集合中的元素。
// 一个管道就是一个聚合操作的序列，比如这里的：filter-map-forEach
list.stream()
    .filter(p->p.getAge() == 23 && p.getGender()==Person.MALE)
    .map(p->p.getEmailAddress())
    .forEach(email->System.out.println(email));
```

#### 在lambda表达式中访问外部作用域中的局部变量
lambda表达式和内部类一样，可以访问包含它们的外部作用域中的局部变量。
但是和内部类不同的是，lambda表达式是词法作用域(lexical scope)，
不会生成新的作用域，所有不存在内部类的同名变量的隐藏的问题。

#### lambda表达式的目标类型
考虑如下代码：

```java
public static void printPersons(List<Person> list, CheckPerson tester);
public static void printPersonWithPredict(List<Person> list, Predict<Person> tester);
```

对于上面两个方法中的tester参数，我们都可以传入一个lambda表达式。Java运行时环境，通过上下文来决定一个lambda表达式的类型，
上列中，lambda表达式的类型就是各自函数所期待的目标类型。lambda表达式能够使用的地方，
就是Java运行时能够通过上下文来确切地决定一个目标类型的地方，这些地方有：
- 变量声明
- 赋值
- 返回语句
- 数组初始化语句(Array initializers)
- 方法和构造器参数
- lambda表达式体
- 条件表达式：? :
- 类型转换表达式(cast expression)

#### lambda表达式的序列化
如果lambda表达式中捕获的局部变量和lambda表达式的目标类型是可以序列化的话，那么lambda表达式就是可以序列化的。
但是和内部类一样，强烈不建议对它们进行序列化。