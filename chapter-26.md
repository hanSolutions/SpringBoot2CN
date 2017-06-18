# 第26章 日志(Logging)

Spring Boot 所有内部日志都使用 [Commons Logging](https://commons.apache.org/proper/commons-logging/)，但开放底层日志实现。为[Java Util Logging](https://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html)，[Log4J2](https://logging.apache.org/log4j/2.x/) 和[Logback](https://logback.qos.ch/) 提供了默认配置。在每个loggers都预先配置使用控制台输出，也可使用可选文件输出。

默认情况下，如果你使用 `Starters`, Logback将用作日志。同时也包含了得当的Logback规则确保依赖库使用 Java Util Logging, Common Logging, Log4J 或SLF4J时均可正常运行。

> Java中有许多可用的日志架构。如果上述列表看起来有点混乱，不用担心。一般情况下你不需要你不需要更改你的日志依赖，Spring Boot默认配置就可以很好地运行。

## 26.1 日志格式

Spring Boot 默认输出如下：

```
2014-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2014-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2014-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2014-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]

```

以下是输出项：

    ＊日期和时间 -- 精确到毫秒， 并且能简单地排序
    ＊Log等级 -- `ERROR`,`WARN`,`INFO`,`DEBUG`,`TRACE`
    ＊进程ID
    ＊分隔符`---`表示日志信息的开始
    ＊线程名－框在方括号中(在控制台输出中常被缩减)
    ＊Logger名－通常是源类名(经常使用缩写)
    ＊日志信息

> Logback没有`FATAL`等级(它映射到`ERROR`)

## 26.2 控制台输出(Console output)

默认日志配置将信息传送到控制台。默认情况下`ERROR`, `WARN`,`INFO`等级的信息会被录入。你也可以在启动应用时加上`--debug`，开启“debug”模式。

```java
$ java -jar myapp.jar --debug
```

> 你也可以在*application.properties*中设定*debug=true*

当开启debug模式时，选择配置核心loggers(嵌入式容器，Hibernate以及Spring Boot)以输出更多信息。开启debug模式*不会*将你的应用设置为输出所有`DEBUG`级别的信息。

或者，你可以在启动应用时加上`--trace`，开启“trace”模式(或在*application.properties*中设定*trace=true*)。将在选择的核心loggers(嵌入式容器，Hibernate结构生成以及整个Spring文件夹)中启用追踪(trace)日志。

## 26.2.1 彩色代码输出

如果你的终端支持ANSI, 输出彩色代码可以辅助阅读。你可以将 `spring.output.ansi.enabled`设置为一个系统支持的值去覆盖自动检测。

颜色编码用`%clr`转换词配置。最简单的格式是根据日志等级配置颜色，比如：

```
%clr(%5p)
```

对应日志等级的颜色如下：

|LEVEL         |Color       |
|--------------|------------|
|`FATAL`       |Red         |
|`ERROR`       |Red         |
|`WARN`        |Yellow      |
|`INFO`        |Green       |
|`DEBUG`       |Green       |
|`Trace`       |Green       |


或者，你可以在转换器中提供选项，用于设定特定的颜色或风格。比如，让文本使用黄色：

```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和风格：

*`blue`
*`cyan`
*`faint`
*`green`
*`magenta`
*`red`
*`yellow`

## 26.3 文件输出

默认情况下，Spring Boot只将日志输出到控制台，不会写入日志文件。除了控制台之外如果你想写入日志文件，你需要设置一个 `logging.file` 或 `logging.path`属性(比如在你的`application properties`).

下表展示了`logging.*`属性如何搭配使用：

### 表 26.1. 日志属性

|`logging.file` |`logging.path` |例子     |说明                                 |
|---------------|---------------|---------|------------------------------------|
|(*none*)|(*none*)|  |只有控制台输出|
|指定文件|(*none*)|`my.log`|写入指定日志文件。名字可以是一个精确位置或相对于当前目录|
|(*none*)|指定目录 |`/var/log`|在指定目录中写`spring.log`。名字可以是一个精确位置或相对于当前目录|

当日志文件大小达到10MB时将与控制台输出同时循环，默认只输出`ERROR`,`WARN` 和`INFO`级别的信息。

> 日志系统在早期的应用生命周期初始化，正因如此，通过@PropertySoource注解载入的属性文件中不会显示日志属性。

> 日志属性独立真正的日志基础架构之外。所以特定的配置主键(比如Logback中的logback.configurationFile)不受Spring Boot管理。

## 26.4 日志等级

所有Spring支持的日志系统都可以在Spring的`Enviroment`中设置日志级别（例如在`application.properties`中设置) 使用 `logging.level.*=LEVEL`设置，其中`LEVEL`可以是 `TRACE`, `DEBUG`, `INFO`, `WARN` ,`ERROR`,`FATAL`,`OFF`中任意一个。root looger可以用`logging.level.root`来配置。例如`application.properties`中：

```java
logging.level.root=WARN
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

> 默认设置中Spring Boot会重新映射Thymeleaf的`INFO`级别信息，这样它们可以输出为`DEBUG`级别信息。这可以帮助减少标准日志输出中的干扰信息。更多如何在你的配置中使用重新映射的信息请参考[LevelRemappingAppender](https://github.com/spring-projects/spring-boot/blob/master/spring-boot/src/main/java/org/springframework/boot/logging/logback/LevelRemappingAppender.java)。

## 26.5 日志配置自定义

通过在类路径中包含适当的库可以激活多种日志系统，此外，将适当的配置文件放在类路径的root中或 Spring `Environment`属性中`logging.config`指定的位置即可对日志系统进一步自定义。

你可以使用系统属性中的`org.springframework.boot.logging.LoggingSystem`来强制Spring Boot使用特定的日志系统。属性中的值应是`LoggingSystem`实现中完全限定的class名。 你也可以通过使用`none`值来彻底禁用Spring Boot的日志配置。

> 因为日志是在`ApplicationContext`创建之前初始化，所以不可能使用Spring `@Configuration` 文件中的`PropertyScources`来控制日志。系统属性以及常见的Spring Boot外部配置文件可以很好的运行。

根据你的日志系统将加载下列文件：

|日志系统      |自定义                                          |
|------------------|------------------------------------------------------------------------------|
|Logback      |`logback-spring.xml`, `logback-spring.groovy`, `logback.xml` 或`logback.groovy`|
|Log4j2       |`log4j2-spring.xml` 或 `log4j2.xml`|
|JDK(Java Util Looging)|`logging.properties`|

> 我们建议尽可能的使用`-spinrg`的变形来设定你的日志配置(比如使用`logback-spring.xml`替代`logback.xml`)。如果你使用标准配置路径，Spring可能无法完全控制日志初始化。

> 在Java Util Logging中运行`executable jar`会引起类加载问题。我们建议你尽可能避免它。

为了帮助自定义其他从Spring `Environment`中转移到系统属性的属性：

|Spring环境       |系统属性       |注解                            |
|-----------------------------|--------------------------------|---------------------------------------------------------------|
|`logging.exception-conversion-word`|`LOG_EXCEPTION_CONVERSION_WORD`|当日志出现异常时使用这个转换词|
|`logging.file`|`LOG_FILE`|如果定义了会在默认日志文件中使用|
|`logging.path`|`LOG_PATH`|如果定义了会在默认日志文件中使用|
|`logging.pattern.console`|`CONSOLE_LOG_PATTERN`|在控制台中使用的日志模式(stdout).(只支持Logback默认设置)|
|`logging.pattern.file`|`FILE_LOG_PATTERN`|在控制台中使用的日志模式(如果启用LOG_FILE).(只支持Logback默认设置)|
|`logging.pattern.level`|`LOG_LEVEL_PATTERN`|
|`PID`|`PID`|目前的进程ID(尽可能找，当没有指定系统环境变量)


当转换日志系统配置文件时，所有的日志系统都支持查询系统属性。比如可以查看`spring-boot.jar`的默认配置。

> 如果你想在日志属性中使用占位符，你应该使用Spring Boot的语法而不是底层框架的语法。尤其当你使用Logback时，应该使用`:`作为属性名与默认值之间的分隔符，而不是`:-`.

> 你可以通过覆盖`LOG_LEVEL_PATTERN`(或Logback中的`logging.pattern.level`)将MDC和其他专门内容加入到日志行中。比如，如果你使用`logging.pattern.level=user:%X{user} %5p`，那么如果MDC存在的话，默认日志格式将包含MDC输入。例如：
> ```
> 2015-09-30 12:30:04.031 user:juergen INFO 22174 --- [  nio-8080-exec-0] demo.Controller
Handling authenticated request
```

## 26.6 Logback扩展

Spring Boot包含大量Logback拓展，这可以帮助你做高级配置。你可以在你的`logback-spring.xml`配置文件中使用这些扩展。

> 你无法再标准`logback.xml`配置文件中使用扩展，因为它过早加载。你应该使用`logback-spring.xml`或定义`logging.config`属性。

> 扩展无法与Logback的配置扫描[configuration scanning](http://logback.qos.ch/manual/configuration.html#autoScan)同时使用。如果你尝试这么做，对配置文件进行修改讲会导致类似以下的错误：
> ```
> ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```

## 26.6.1 文件特定配置

`<springProfile>`标签允许你选择性的包含或排除基于激活Spring文件的部分配置。再任何含有`<configuration>`的地方都支持文件部分。使用`name`属性来指定接受配置的文件。多个文件可以用一个逗号隔开的列表制定。

```
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev, staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

## 26.6.2 环境属性

`<springProperty>`标签允许你在Logback使用从Spring `Environment`中获取的属性。如果你想从你的`application.properties`文件中读取Logback配置，这个标签将会非常有用。这个标签与Logback的标准`<property>`标签相似，但取代制定一个直接的`value`，你(从`Environment`中)指定属性中的`source`。如果你需要在某个除`local`之外的地方存储属性，你可以使用`scope`属性。如果你需要一个备用值以防属性没有设置在`Environment`中，你可以使用`defalutValue`属性。

```
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

> `RelaxedPropertyResolver`用于访问`Environment`属性。如果在破折号中指定`source`(my-property-name)，所有的近似变化都会被尝试一遍(myPropertyName,MY_PROPERTY_NAME 等等)。


























