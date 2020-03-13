## Log4j的使用模式

### 基本使用方式
使用自带的实现，基本步骤如下：
1. 引入Log4j的jar，包括API和实现。

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.13.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.13.1</version>
    </dependency>
</dependencies>
```

2. 在代码中使用Logger对象来产生日志事件。典型的方式是，每个类一个静态的Logger。

```java
public class A {
    // 不传参的话，使用的是当前的类作为参数
    private static Logger logger = LogManager.getLogger();
}
```

3. 设计配置文件[参考](log4j)