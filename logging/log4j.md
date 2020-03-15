## Log4j 2特定及其配置

[参考](http://logging.apache.org/log4j/2.x/manual/configuration.html)

Log4j目前已经发展到Log4j 2.x了，1.x的Log4j已经完成了它的历史使命。最新的Log4j也叫Log4j 2。

### Log4j 2的特性

- API分离，Log4j 2的API是和它的实现分离的。使用者只使用API中定义的类和方法。而其实现可以持续的改进，保证前向兼容。
Log4j 2 API就是一个日志门面，除了可以和Log4j的实现一起使用外，也可以和其他的实现一起使用，比如Logback。
- 性能优异，包含了基于LMAX Disruptor library的下一代异步Logger。比JUL，Logback，Log4j 1.x性能好很多。
- Log4j 2 API支持多种其他的日志框架，如：JUL，Common Logging，Log4j 1.2， SLF4J。
- 使用Log4J 2 API，通过log4j-to-slf4j的适配包，可以使用任何与SLF4J兼容的库作为它的实现。
- 自动加载配置，和Logback一样，如果配置被修改了，可以自动重新加载。比Logback做的好的就是在重新加载配置的过程中，不会丢失日志。
- 高级过滤
- 插件架构，Log4j使用插件模式来配置它的组件，因此，你不需要写代码来创建和配置Appender，Layout，Pattern，Converter等等。
- Property属性替换支持，属性可以来源于配置文件，环境变量，系统变量，日志时间里的数据等等。
- Java 8的lambda支持，如果相应级别的日志没有开启，那么lambda不会求值。
- 可以在代码或者配置文件中自定义的日志级别，不需要继承别的类。
- 日志Builder API，除了通过各种日志方法来生成日志事件外，日志事件可以通过一个Builder来生成。
- 免垃圾回收，在稳定状态记录日志时，Log4j在独立的应用中是没有垃圾回收的，在Web应用中只有很少的垃圾回收。
- 与应用服务器集成，从2.10.0开始，增加了log4j-appserver模块来支持与Apache Tomcat和Eclipse Jetty的集成。
- 云支持，从2.12.0开始，通过Lookup支持访问Docker容器信息，通过Spring Cloud的配置来访问和修改Log4j的配置。2.13.0开始支持访问Spring Boot配置信息和K8S的信息。

### Log4j的配置

有4中配置方式：
- 通过配置文件，可以是XML, JSON, YAML, properties格式
- 编程方式，通过创建ConfigurationFactory哈Configuration
- 编程方式，通过调用Configuration接口暴露的API，给默认的配置增加组件
- 编程方式，通过调用内部的Logger类上的方法

#### 自动配置

Log4j能够在应用初始化的时候，自动配置好自己。Log4j会定位到类路径下所有的ConfigurationFactory插件，按照权重从高到低排好序。
Log4j自带有4个ConfigurationFactory的插件实现，分别对应四种不同格式的配置文件。
如果使用json格式，那么必须包含下面依赖：
```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.10.3</version>
</dependency>
```
如果使用yaml格式，那么必须包含下面依赖：
```xml
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-yaml</artifactId>
  <version>2.10.3</version>
</dependency>
```

Log4j的自动配置过程如下：
- 系统属性log4j.configurationFile存在，使用与该文件扩展名匹配的ConfigurationFactory加载该文件的配置。
- 不存在上述系统属性，类路径是否存在properties格式文件：log4j-test.properties
- 不存在properties文件，是否存在YAML格式配置文件：log4j-test.yaml或者log4j-test.yml
- 不存在YAML文件，是否存在JSON格式配置文件：log4j-test.json或者log4j-test.jsn
- 不存在JSON文件，是否存在XML格式配置文件：log4j-test.xml
- 如果-test形式的配置文件都不存在，按照上面的顺序判断不带-test的文件是否存在。
- 如果都不存在，使用ConfigurationFactory的默认实现类DefaultConfiguration，意味着所有日志打印到控制台。

#### 自动刷新配置

如果是通过配置文件配置的，那么当配置文件有修改时，Log4j会自动重新加载配置文件，进行刷新，
并且刷新期间不会丢失日志事件。

#### 配置文件语法（以XML格式为例）

##### 根元素configuration
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration
    advertiser="multicastdns"  
    dest="指定目标，可以是err, out, filepath, url"
    monitorInterval="每隔多少秒检查一次配置文件是否被修改"
    name="取一个名字"
    packages="在哪些包下去发现插件"
    schema="strict模式下，用来验证配置文件的schema"
    shutdownHook="是否会在JVM shutdown的时候shutdown Log4j，默认是会，设为disable时不会"
    shutdownTimeout="多少毫秒用来shutdown Log4j，设的太低的话，可能丢失日志"
    status="Log4j内部日志的级别，设为trace可以输出Log4j内部的详细日志，用来定位问题，设置系统变量log4j.debug也可以用来输出内部详细日志"
    strict="启用XML的strict模式，对于其他的格式不支持这个属性"
    verbose="开启时，会输出加载插件的详细日志"
></Configuration>
```

##### 整体结构
xml配置有精简模式和严格模式两种。经过测试发现，严格模式下，也可以使用精简模式的配置。但是精简模式下，不能使用严格模式的配置。

精简模式下，一些组件直接使用简单的元素就可以表示，比如Console的Appender，
直接使用Console元素就可以。**元素以及属性的名字不区分大小写**。

严格模式下可以使用schema来进行验证，不过写起来就比较繁琐一些。比如Console的Appender，
必须使用Appender元素，指定type=Console等等。

精简的例子：
```xml
<?xml version="1.0" encoding="UTF-8"?>;
<Configuration>
  <Properties>
    <Property name="name1">value</property>
    <Property name="name2" value="value2"/>
  </Properties>
  <filter  ... />
  <Appenders>
    <appender ... >
      <filter  ... />
    </appender>
    ...
  </Appenders>
  <Loggers>
    <Logger name="name1">
      <filter  ... />
    </Logger>
    ...
    <Root level="level">
      <AppenderRef ref="name"/>
    </Root>
  </Loggers>
</Configuration>
```

严格的例子
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="debug" strict="true" name="XMLConfigTest"
               packages="org.apache.logging.log4j.test">
  <Properties>
    <Property name="filename">target/test.log</Property>
  </Properties>
  <Filter type="ThresholdFilter" level="trace"/>
 
  <Appenders>
    <Appender type="Console" name="STDOUT">
      <Layout type="PatternLayout" pattern="%m MDC%X%n"/>
      <Filters>
        <Filter type="MarkerFilter" marker="FLOW" onMatch="DENY" onMismatch="NEUTRAL"/>
        <Filter type="MarkerFilter" marker="EXCEPTION" onMatch="DENY" onMismatch="ACCEPT"/>
      </Filters>
    </Appender>
    <Appender type="Console" name="FLOW">
      <Layout type="PatternLayout" pattern="%C{1}.%M %m %ex%n"/><!-- class and line number -->
      <Filters>
        <Filter type="MarkerFilter" marker="FLOW" onMatch="ACCEPT" onMismatch="NEUTRAL"/>
        <Filter type="MarkerFilter" marker="EXCEPTION" onMatch="ACCEPT" onMismatch="DENY"/>
      </Filters>
    </Appender>
    <Appender type="File" name="File" fileName="${filename}">
      <Layout type="PatternLayout">
        <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
      </Layout>
    </Appender>
  </Appenders>
 
  <Loggers>
    <Logger name="org.apache.logging.log4j.test1" level="debug" additivity="false">
      <Filter type="ThreadContextMapFilter">
        <KeyValuePair key="test" value="123"/>
      </Filter>
      <AppenderRef ref="STDOUT"/>
    </Logger>
 
    <Logger name="org.apache.logging.log4j.test2" level="debug" additivity="false">
      <AppenderRef ref="File"/>
    </Logger>
 
    <Root level="trace">
      <AppenderRef ref="STDOUT"/>
    </Root>
  </Loggers>
 
</Configuration>
```

#### 配置Logger
要正确的配置Log4j，必须理解Log4j里面的一些概念。比如：logger, appender, filter, layout, marker等

Logger通过logger元素<logger>进行配置。代表一个LoggerConfig。

logger必须包含一个name属性，还可以包含level属性，以及additivity属性。
level别是日志级别，可以是TRACE, DEBUG, INFO, WARN, ERROR, FATAL中的一个，默认为ERROR。
additivity表示日志的可传递性，可以为true或者false，默认为false。

一个Log4j的配置必须指定一个RootLogger，如果不指定，会使用一个默认的，把日志输出到控制台的Logger。

如：
```xml
<Logger name="com.winson.Hello" level="TRACE" additivity="false"/>
```

#### 配置Appender
Appender是实际处理日志事件的插件。

Appender可以通过插件的名字来配置，也可以通过appender元素加一个type属性来配置，type属性表示插件的名字。

每个Appender必须由一个唯一的名字, Logger对象可以通过这个名字进行引用。

大部分的Appender还支持配置一个Layout插件对象，和Appender插件对象一样，该插件对象可以通过插件名来指定或者通过layout元素和一个type属性来指定，type属性是插件的名字。

不同的Appender插件，还可能会定义其他不同的属性。

如：
```xml
<appender type="Console" name="console"/> 或者 <Console name="console"/>
```

#### 配置Filter
Filter是对日志事件进行过滤的插件对象。Log4j可以在四个地方来配置Filter。
1. 顶级的，也就是和Loggers, Appenders, Properties元素同一级别定义，这些filter可以在把事件传递到具体的logger之前接受或者拒绝事件。
2. 在Logger元素内部定义，可以对特定的Logger进行日志事件的接受或者拒绝。
3. 在Appender元素内部定义，可以在Appender处理日志之前，对日志进行接受或者拒绝。
4. 在一个Appender引用元素上，可以决定一个Logger是否把日志事件路由到Appender。

一般来说，一个元素只能配置一个filter，但是我们可以使用filters元素来配置多个filter，形成一个组合Filter。
例如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="debug" name="XMLConfigTest" packages="org.apache.logging.log4j.test">
  <Properties>
    <Property name="filename">target/test.log</Property>
  </Properties>
  <!-- 1.顶级的Filter -->
  <ThresholdFilter level="trace"/>
  <Appenders>
    <Console name="STDOUT">
      <PatternLayout pattern="%m MDC%X%n"/>
    </Console>
    <Console name="FLOW">
      <PatternLayout pattern="%C{1}.%M %m %ex%n"/>
      <!--3.Appender元素内部，并且是一个组合Filter-->
      <filters>
        <MarkerFilter marker="FLOW" onMatch="ACCEPT" onMismatch="NEUTRAL"/>
        <MarkerFilter marker="EXCEPTION" onMatch="ACCEPT" onMismatch="DENY"/>
      </filters>
    </Console>
    <File name="File" fileName="${filename}">
      <PatternLayout>
        <pattern>%d %p %C{1.} [%t] %m%n</pattern>
      </PatternLayout>
    </File>
  </Appenders>
  <Loggers>
    <Logger name="org.apache.logging.log4j.test1" level="debug" additivity="false">
      <!--2.Logger元素内部定义，对当前Logger的日志事件进行过滤-->
      <ThreadContextMapFilter>
        <KeyValuePair key="test" value="123"/>
      </ThreadContextMapFilter>
      <AppenderRef ref="STDOUT"/>
    </Logger>
 
    <Logger name="org.apache.logging.log4j.test2" level="debug" additivity="false">
      <Property name="user">${sys:user.name}</Property>
      <AppenderRef ref="File">
        <!--4.在AppenderRef内部定义-->
        <ThreadContextMapFilter>
          <KeyValuePair key="test" value="123"/>
        </ThreadContextMapFilter>
      </AppenderRef>
      <AppenderRef ref="STDOUT" level="error"/>
    </Logger>
 
    <Root level="trace">
      <AppenderRef ref="STDOUT"/>
    </Root>
  </Loggers>
 
</Configuration>
```

#### 属性替换
Log4j支持在配置文件中使用一个占位符来引用在别处定义的属性。这些属性的解析可能是在解析配置文件的同时就进行了解析，也可以是在运行时把它传递给具体的组件来进行解析。占位符通过${name}的形式来声明。

更高级的是，Log4j还支持${context:name}这种形式的占位符，context指定name的上下文。Log4j内置的context有：
- base64: name是一个base64编码的数据，如：{base64:SGVsbG8gV29ybGQhCg==}表示是Hello world.
- bundle: 资源包，其中name必须遵循包的命名约定，如{bundle:com.winson.Message:MyKey}
- ctx: 线程上下文Map （MDC）
- date： 插入当前日期时间，name表示采用的格式化方式。
- env： 系统环境变量，{env:ENV_NAME}或者{env:ENV_NAME:-defalut_value}
- jndi: jndi中的变量值
- jvmrunargs: 
- log4j: log4j配置文件的信息，如{log4j:configLocation}表示配置文件绝对路径，{log4j:configParentLocation}表示配置文件所在的目录。
- main: 通过MapLookup.setMainArguments(String[])设置的值
- map: MapMessage中的值
- sd: StructuredDataMessage中的一个值
- sys: 系统属性，{sys:some.property}或者{sys:some.property:-default_value}

通过顶级的Properties元素定义的属性为默认属性，在占位符解析不出值的时候，使用默认值。

以多个$符号开头的占位符，在每次解析的时候，会去掉一个$符号。第一次解析是解析配置文件。

#### 配置中的脚本
Log4j在一些组件中，还支持使用脚本，支持的语言包括：JavaScript, Groovy, Beanshell。

#### 使用XInclude来包含其他配置文件
例如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration xmlns:xi="http://www.w3.org/2001/XInclude"
               status="warn" name="XIncludeDemo">
  <properties>
    <property name="filename">xinclude-demo.log</property>
  </properties>
  <ThresholdFilter level="debug"/>
  <xi:include href="log4j-xinclude-appenders.xml" />
  <xi:include href="log4j-xinclude-loggers.xml" />
</configuration>
```

#### 组合配置文件
可以通过log4j.configurationFile系统属性指定多个log4j的配置文件，Log4j可以以一定的规则合并它们。

#### 状态信息
```xml
<Configuration status="trace/debug/info/...">
log4j.debug
```
这些配置可以控制Log4j的内部日志，对应定位日志问题很有帮助。

#### 测试环境的配置
1. 通过在测试资源路径下包含-test结尾的配置文件，可以保证使用的是测试的配置。
2. 在JUnit测试类里通过@BeforeClass，使用log4j.configurationFile来指定测试的配置。
3. 在Junit测试类使用LoggerContextRule。

#### 系统属性
Log4j使用了很多的系统属性来控制它的行为，这些系统属性有以下的优先级：

环境变量 > 属性文件 > 常规系统属性

1. 所有的环境变量以LOG4J_开头，全部大写，以下划线连接。
2. 属性文件指的是类路径下的这个文件：log4j2.component.properties。
3. 常规系统属性也就是-Dname=value指定的属性，优先级最低，可被其他两种来源的属性覆盖。

#### JSON配置方式
```json
{
  "configuration": {
    "status": "debug",
    "properties": {
      "property": {"name": "filename", "value": "/usr/local/test.xml"},
      "property": {"name": "username", "value": "Steven Jobs"}
    },
    "ThreholdFilter": {"level", "debug"},
    "appenders": {
      "Console": {
        "name": "STDOUT",
        "PatternLayout": {"pattern": "???%m%n"}
      },
      "appender": [
        {
          "type": "Console", 
          "name": "STDOUT1", 
          "PatternLayout": {
            "pattern": "===%m%n"
          }
        }
      ]
    },
    "loggers": {
      "logger": [
        {"name": "EventLogger", "level": "debug", "additivity": "false"},
        {"name": "AnotherLogger", "level":"warn", "appenderRef": {"ref": "STDOUT1"}}
      ],
      "root": {"level":"trace", "appenderRef": {"ref":"STDOUT"}}
    }
  }
}
```

#### properties配置方式
```properties
status = error
dest = err
name = PropertiesConfig
 
property.filename = target/rolling/rollingtest.log
 
filter.threshold.type = ThresholdFilter
filter.threshold.level = debug
 
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %m%n
appender.console.filter.threshold.type = ThresholdFilter
appender.console.filter.threshold.level = error
 
appender.rolling.type = RollingFile
appender.rolling.name = RollingFile
appender.rolling.fileName = ${filename}
appender.rolling.filePattern = target/rolling2/test1-%d{MM-dd-yy-HH-mm-ss}-%i.log.gz
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = %d %p %C{1.} [%t] %m%n
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy
appender.rolling.policies.time.interval = 2
appender.rolling.policies.time.modulate = true
appender.rolling.policies.size.type = SizeBasedTriggeringPolicy
appender.rolling.policies.size.size=100MB
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.max = 5
 
logger.rolling.name = com.example.my.app
logger.rolling.level = debug
logger.rolling.additivity = false
logger.rolling.appenderRef.rolling.ref = RollingFile
 
rootLogger.level = info
rootLogger.appenderRef.stdout.ref = STDOUT
```

#### YAML配置方式
```yaml
Configuration:
  status: warn
  name: YAMLConfigTest
  properties:
    property:
      name: filename
      value: target/test-yaml.log
  thresholdFilter:
    level: debug
  appenders:
    Console:
      name: STDOUT
      PatternLayout:
        Pattern: "%m%n"
    File:
      name: File
      fileName: ${filename}
      PatternLayout:
        Pattern: "%d %p %C{1.} [%t] %m%n"
      Filters:
        ThresholdFilter:
          level: error
 
  Loggers:
    logger:
      -
        name: org.apache.logging.log4j.test1
        level: debug
        additivity: false
        ThreadContextMapFilter:
          KeyValuePair:
            key: test
            value: 123
        AppenderRef:
          ref: STDOUT
      -
        name: org.apache.logging.log4j.test2
        level: debug
        additivity: false
        AppenderRef:
          ref: File
    Root:
      level: error
      AppenderRef:
        ref: STDOUT
          
```