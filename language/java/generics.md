### 泛型

泛型，就是在定义类，接口，方法的时候，使用类型（类，接口）作为参数。

使用泛型的好处：
- 编译时进行强类型检查
- 避免类型转换
- 实现通用的泛型算法，集合框架用的比较多

经常使用的泛型的类型参数命名约定：
- E - Element （被Java的集合框架大量使用）
- K - Key
- N - Number
- T - Type
- V - Value
- S, U, V etc - 第二个，第三个，第四个类型参数


#### 泛型类和接口
使用如下的方式来定义泛型类和接口
```java
public class ClassName<T1, T2, ..., Tn> { }
public class InterfaceName<T1, T2, ..., Tn> {}
```
其中的T1, T2, ..., Tn表示泛型的类型参数。类型参数表示一个类，接口，或者其他的泛型定义。

比如定义一个盒子泛型类：
```java
public class Box<T> {
    private T t;
    public void set(T t) {
        this.t = t;
    }
    public T get() {
        return this.t;
    }
}

// 使用具体的类型作为实际参数，声明一个泛型类对象的引用：
Box<Integer> integerBox;
// 实例化一个泛型类对象：
integerBox = new Box<Integer>();
// 从Java SE 7开始，编译器有了类型推断功能，实例化的时候，可以省略类型参数：
Box<Integer> box = new Box<>();
```

##### 泛型类的原始类型
泛型的原始类型，就是不带类型参数的泛型的名字。比如上面的Box就是`Box<T>`的原始类型
```java
// 可以使用泛型的原始类型，如：
Box rawBox = new Box(); // 此时Box中存的类型是Object类型
```

你可能在历史遗留代码中可以看到原始类型，因为在Java SE5之前都没有泛型的概念。

为了向后兼容，把一个泛型对象，赋值给一个原始类型的引用是可以的：
```java
Box<String> stringBox = new Box<>();
Box rawBox = stringBox; // 这样赋值是没问题的
```

但是，如果把一个原始类型的对象，赋值给一个泛型对象的引用，编译器会提示一个uncheck的警告：
```java
Box rawBox = new Box(); // rawBox是Box<T>的原始类型
Box<Integer> intBox = rawBox; // 警告：未被检查的类型转换
```

所以，我们应该尽量避免使用泛型的原始类型。


#### 泛型方法

包含了自己的类型参数（不属于类或者接口的类型参数）的方法，就是泛型方法。
方法的类型参数作用域只限于定义它的方法。静态和非静态方法都可以是泛型方法。
类的构造器也可以是泛型的。

定义与调用方式举例如下，其中：
- 定义时，把类型参数放在方法的返回值类型前
- 调用时，类型参数需要放到点操作符(.)和方法名中间

```java
// 类型参数放置于方法的返回值类型前，如下，放在boolean前
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
                p1.getValue().equals(p2.getValue());
    }
}
public class Pair<K, V> {
    private K key;
    private V value;
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey() { return this.key; }
    public V getValue() { return this.value; }
}

// 调用泛型方法的完整语法如下，泛型需要放置在点号与方法名中间
// 当然有了类型推断后，调用时的类型参数，也是可以省略的
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean isSame = Util.<Integer, String>compare(p1, p2);
// 省略类型参数，进行调用
boolean same = Util.compare(p1, p2);
```

#### 有界的类型参数
有时候，你需要对类型参数的类型进行限制，比如一个操作数字的方法，只能接受Number及其子类的实例作为参数。
通过有界的类型参数，就可以实现这个需求。


声明有界类型参数的方式：
1. 在类型参数后使用extends 再后接类型（类或者接口）来指定类型参数的上界
```java
// 上界是Integer, 表示类型参数T只能是Integer或者它的子类
public class NatureNumber<T extends Integer> {}
// 上界也可以是接口，表示实现了这个接口
public class Executor<T extends Runable> {}
// 上界可以有多个，多个使用 & 连起来，
// 由于类是单继承，多实现，所以最多只能是一个类，并且类必须放到第一个
public class D <T extends A & B & C> {}
```

#### 有界类型参数和泛型方法
有界类型参数，是实现泛型算法的一个关键。比如我们要统计一个数组中，大于某个元素的个数：
```java
public static <T> int count(T[] array, T element) {
    int count = 0;
    for (T t: array) {
        if ( t > element) { // 编译错误，因为 > 操作符只能用于原始数据类型上(int, long, float ...)
            count++;
        }
    }
    return count;
}
```

为了解决这个问题，我们可以使用有界类型参数，来指定T的范围：
```java
public interface Compare<T> {
    public int compareTo(T t);
}
// 指定T是实现了Compare接口的类
public static <T extends Compare<T>> int count(T[] array, T element) {
    int count = 0;
    for (T t: array) {
        if (t.compare(element) > 0) {
            count++;
        }
    }
    return count;
}
```

#### 泛型的继承，和子类型
面向对象中，只要两个类型是兼容的，那么可以把一个类型的对象，赋值给另一个类型的引用。
```java
Object someObject = new Object();
Integer someInt = new Integer(10);
someObject = someInt; // 这样没问题

public void someMethod(Number n) {}
someMethod(new Integer(10)); // 没问题，因为Integer是Number的子类
someMethod(new Double(10.1)); // 没问题，因为Double也是Number的子类型

// 但是对于泛型，需要特别注意，不要引起混淆
// 虽然Integer是Number的子类型，但是Box<Integer>不是Box<Number>的子类型
public void someMethod(Box<Number> numberBox) {}
someMethod(new Box<Integer>(10)); // 编译错误
```

泛型类或者接口的子类型必须是通过extends从句来决定的，以集合类为例：
`ArrayList<E>`实现了`List<E>`, `List<E>`继承了`Collection<E>`，所以：
- `ArrayList<String>`是`List<String>`的子类型
- `List<String>`是`Collection<String>`的子类型

我们自己定义了一个泛型接口：
```java
interface PayloadList<E, P> extends List<E> {
    void setPayload(int index, P val);
}
```

那么如下的类，都是List<String>的子类型，因为他们的接口具有继承关系，同时类型参数也是一样的：
- PayloadList<String, String>
- PayloadList<String, Integer>
- PayloadList<String, Exception>

#### 类型推断
类型推断是编译器的一种能力，能够根据函数的参数类型或者返回值类型，来推断函数调用时，传入的参数应该是什么数据类型。
如果有多种类型都符合的话，推断算法，可以推断出最确切的数据类型。
```java
// 比如有如下方法定义和调用
static <T> T  pick(T t1, T t2) {return t2;}
Serializable s = pick("a", new ArrayList<String>());
// 根据返回值类型，可以推断出，pick调用时，两个参数类型都应该是Serializable类型
```

#### 泛型类的实例化
```java
// 可以省略类型参数
Map<String, Object> map = new HashMap<>(); // 省略了
Map<String, Object> map = new HashMap(); // 会有警告，必须使用<>来开启类型推断功能
```

#### 构造器的泛型化
泛型类和非泛型类的构造器本身，也是可以泛型化的，也就是像泛型方法一样，引入自己的类型参数。
```java
// 类是泛型的，包含类型参数T
class MyClass<T> {
    public <U> MyClass(U u) {
        //此构造器也是泛型的，包含类型参数U
    }
}

MyClass<Integer> myClass = new MyClass<>(""); // 通过构造器，推断出U的类型为String
```

#### 泛型中的wildcard（通配符）：?
泛型代码中，统配符就是一个问号（？），表示类型是不确定的。
统配符可以使用在以下情形中：
- 作为field，方法的参数，局部变量的类型
- 有时候返回值类型也使用通配符（但是最好要避免这种情况）

但是在泛型方法调用时，泛型类实例化时，或者继承一个其他的泛型类时，传入的类型参数不能包含通配符。

##### 固定上界通配符
使用extends来指定上界类型，表示？所代表的类型只能是上界及其子类型
```java
public static void sumOfList(List<? extends Number> list) {
    // ? 表示list中的元素是Number类型，及其子类型都可以
    double s = 0.0;
    for (Number n: list) {
        // n 类型是 Number
        s += n.doubleValue();
    }
}
// 下面两种参数类型都可以传递给这个函数
List<Integer> intList = Arrays.asList(1, 2, 3);
System.out.println("sum = " + sumOfList(intList)); // sum = 6.0
List<Double> doubleList = Arrays.asList(1.2, 2.3, 3.5);
 System.out.println("sum = " + sumOfList(doubleList)); // sum = 7.0
```

##### 无界通配符
单纯的一个问号，是无界通配符，表示类型不确定。两种常见下使用无界通配符比较有用：
- 如果你实现的方法中，只用到了Object类提供的方法。
- 当你的方法实现中，不需要依赖具体的类型参数，比如List.size, List.clear，都不需要知道具体的类型。
事实上，Class<?>就经常被使用，因为Class<T>中的很多方法，就不依赖于具体的T是什么数据类型。

举例如下：
```java
// 你可能是想实现一个打印任何对象列表的方法，所有使用的是List<Object>
// 但是这个函数只能接受List<Object>类型的参数，
// List<Integer>, List<Number>都不行，因为他们不是List<Object>的子类型
public static void printList<List<Object> list> {
    for (Object elem: list) {
        System.out.println(elem + " ");
    }
    System.out.println();
}

// 这个时候，我们就可以使用无界通配符
// 因为它的实现，只依赖了Object, 和list中是什么数据类型无关
public static void printList<List<?> list> {
    for (Object elem: list) {
        System.out.println(elem + " ");
    }
    System.out.println();
}
// 之后，我们可以传入任意类型的List对象
List<Integer> li = Arrays.asList(1, 2, 3);
List<String> ls = Arrays.asList("one", "two", "three");
printList(li);
printList(ls);
```

值得注意的是，List<Object>和List<?>是有很大不同的。你可以插入任意的对象到List<Object>,
但是，你只能插入null到List<?>，因为里面的类型是不确定的，虽然是不确定的，但是对于任何的引用对象，
都是可以使用null来表示一个空值。

##### 固定下界的通配符
和固定上界的通配符相反，使用super来指定下界类型，表示？所代表的类型只能是下界及其父类型。

假如，你要实现一个方法，把一些Integer类型对象插入到一个List中，
此时为了增加灵活性，List可以List<Integer>,
List<Number>, List<Object>, 这些List对象都可以用来存储一个Integer类型对象。
这种场景下，你就可以使用下界通配符，来限制传入的List中的对象类型是Integer或者它的父类型。
```java
public static void addNumbers(List<? super Integer> list) {
    for (int i=0; i < 10; i++) {
        list.add(i); 
    }
}
// 下面三种参数类型addNumbers都是可以接受的
addNumbers(new ArrayList<Integer>());
addNumbers(new ArrayList<Number>());
addNumbers(new ArrayList<Object>());
```

##### 通配符和子类型
我们已经知道了,虽然Integer是Number的子类型，但是List<Integer>和List<Number>是不存在父子关系的。
事实上，List<Integer>和List<Number>是没有直接关系的，不过他们是可以通过通配符联系起来的，
最简单的联系方式就是：他们都是List<?>的子类型。

更详细的关系说明参考[这里](https://docs.oracle.com/javase/tutorial/java/generics/subtyping.html)

##### 通配符捕获（wildcard capture）与helper方法
```java
import java.util.List;

public class WildcardError {
    // 编译器发现在调用List.set(i, E)时，不能推断出E所表示的具体类型，所以编译器在编译的时候会报错
    // 报错的信息大概是说，set函数期望的类型为int, CAP#1，也就是int型和捕获的类型，
    // 但是实际上给set函数传递的类型为int, Object类型，Object不能转换为CAP#1类型，所以就会报错
    void foo(List<?> i) {
        i.set(0, i.get(0)); // 编译错误
        fooHelper(i); // 使用helper，编译通过
    }

    // 解决上面的问题，就是引入一个helper方法
    // 按约定，helper方法一般命名为：原始方法+Helper
    private <T> void fooHelper(List<T> l) {
        l.set(0, l.get(0));
    }

    // 但是下面这种情况是没法通过helper方法解决的
    void swapFirst(List<? extends Number> l1, List<? extends Number> l2) {
        Number temp = l1.get(0);
        // 期望一个CAP#1 extends Number
        // 实际上传递的是CAP#2 extends Number
        // 他们有相同的上界，但是类型可能不一样，所以编译错误
        l1.set(0, l2.get(0));

        // 期望一个CAP#1 extends Number
        // 实际传递的是Number，但是来自另一个list，也可能数据类型不一样
        l2.set(0, temp);
    }

    // 显然，下面两个对象都满足swapFirst函数的参数要求，
    // 但是他们是不能交换第一个元素的
    List<Integer> li = Arrays.asList(1,2,3);
    List<Double> ld = Arrays.asList(1.0, 2.0, 3.0);
    swapFirst(li, ld); 
}
```

#### 泛型的类型擦除（Type Erasure）
JDK5开始引入的泛型，提供了编译时的类型检查。泛型的类型参数只是在编译期存在，
编译后的字节码中是不存在类型信息的。
例如：`ArrayList<A>`, `ArrayList<B>`两个类，编译后只存在一份字节码，他们的类型信息被擦除了，过程如下：

- 如果类型参数是有界的，类型参数直接替换为上界或者下界，如果是无界的，替换为Object。所以字节码中只包含正常的类型，接口，方法。
- 在需要的地方，插入类型转换代码，保证类型安全。
- 在泛型继承的情况下，生成[桥接方法](https://docs.oracle.com/javase/tutorial/java/generics/bridgeMethods.html)来保证多态。

#### 使用泛型存在的限制
1. 不能使用原始原始类型类初始化一个泛型，如：`List<int>`是不行的。
2. 不能通过new创建类型参数的实例对象，如：
```java
// 泛型方法，包含一个类型参数E
public static <E> void append(List<E> list) {
    E element = new E(); // 编译错误
    list.add(element);
}
// 可以通过反射来创建一个对象
public static <E> void appendList(List<E> list, Class<E> cls){
    E ele = cls.newInstance(); // 这是可以的
    list.add(ele);
}
```
3. 不能声明类型参数对应的静态字段
```java
public class MobileDevice<T> {
    private static T os; 
    // T 不能是静态的
    // 否则, MobileDevice<Smartphone>, MobileDevice<TabletPC>
    // 静态的T，不知道应该是Smartphone还是其他的。
}
```
4. 不能使用instanceof和强制类型转换
```java
public static <E> void rtti(List<E> list) {
    if (list instanceof ArrayList<Integer>) { // 编译时错误，因为类型被擦除了
        //...
    }
}
```
5. 不能创建泛型类型数组
```java
List<Integer>[] arrayOfLists = new List<Integer>[2]; // 编译时错误
```

