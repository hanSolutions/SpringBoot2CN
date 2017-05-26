# 第15章 配置类

Spring Boot倾向于以Java为基础的配置.虽然你也可以使用`SpringApplication`配合XML,我们还是会建议你使用一个`@Configuration`类作为配置的源文件.在大部分情况下main方法所在的类,也是放置主要`@Configuration`的地方.

>许多Spring在网上的已发表的配置示例都使用了XML配置.如果可以,请尽可能使用以Java为基础的配置.搜索`Enable*`注释是一个很好的开始点.

## 15.1 引入附加的配置类

你不需要把你所有的`@Configuration`都放进一个类.`@Import`注释可以用来引入附加的配置类.或者你也可以使用`@ComponentScan`来自动扫描并且识别所有的Spring部件,包括`@Configuration`类.

## 15.2 引入XML配置

如果你必须使用以XML为基础的配置,我们建议你仍然从使用`@Configutarion`类开始.你可以在这之后使用一个附加的`@ImportResource`注释来读取XML配置文件.