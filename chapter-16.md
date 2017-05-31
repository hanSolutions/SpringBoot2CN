## 第16章 自动配置

Spring Boot自动配置试图基于你已经添加的jar依赖来自动配置你的Spring应用程序，例如，如果`HSQLDB`在你的类路径而且你还没有手动设置任何数据库连接beans, 那么我们会自动设置一个内存数据库。

你需要通过添加`@EnableAutoConfiguration`或者`@SpringBootApplication`注解到你的`@Configuration`类的其中一个选择加入自动设置。
>你应该永远只添加一个`@EnableAutoConfiguration`的注解。我们一般建议你添加到你的主要的`@Configuration`类。

## 16.1 逐渐的取代自动配置

自动配置是非侵入性的，在任何时候你可以开始定义你自己的配置来取代自动配置的特定的部分。例如，如果你添加你自己的`DataSource`bean，默认的嵌入数据库支持会退出。

如果你需要找出什么样的自动配置正在且为什么被使用，用`--debug`启动你的应用程序开关。这将对精选的核心日志开启调试日志，同时记录自动配置报告到控制台。

## 16.2 禁止特定的自动配置

如果你发现某个正在被使用的特定的自动配置不是你想要的，你可以使用`@EnableAutoConfiguration`的排除属性来禁止它们。

```
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

如果类不在类路径，你可以使用注解的`excludeName`属性并指定完全合格的名字。最后，你也可以通过`spring.autoconfigure.exclude`属性来控制自动配置类的列表操作排除。

>你可以在注解层面和使用属性来定义可排出类。

