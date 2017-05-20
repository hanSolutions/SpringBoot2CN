#第20章 开发者工具
Spring Boot包含了一些额外的工具集，用于提升体验Spring Boot应用的开发乐趣。`spring-boot-devtools`模块可以被包含到任何模块中来提供development-time特性，你只需简单的将该模块的依赖添加到你的构建中：

**Maven**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```
**Gradle**
```properties
dependencies {
    compile("org.springframework.boot:spring-boot-devtools")
}
```

当执行一个完整的，打包过的应用时，开发者工具（devtools）会被自动禁用。如果你的应用是使用`java -jar`或特殊的类加载器启动，都会被认为是一个产品级的应用（production application），从而禁用开发者工具。设置该依赖级别为optional从而防止devtools传递到项目中的其他模块是个好经验（Best Practice）。由于Gradle不支持`optional`依赖，所以你可能同时要了解下[propdeps-plugin](https://github.com/spring-projects/gradle-plugins/tree/master/propdeps-plugin)。

由于再打包的压缩应用文件（repackaged archives）默认不包含开发者工具。如果想使用开发者工具的[一些远程特性](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-devtools-remote),你需要禁用`excludeDevtools`构建属性来添加它进应用，Maven和Gradle都支持该属性。


##20.1 默认属性

Spring Boot支持的一些库（libraries）使用缓存提高性能，比如，[模板引擎（template engines）](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-spring-mvc-template-engines)会缓存以编译的模版来避免反复地解析模版文件。当要提供静态资源（文件）时，Spring MVC还能够添加HTTP缓存头字段

虽然缓存会给实际产品带来很大的好处，但它会降低开发效率在产品的开发阶段。它会妨碍你对你的程序修改结果及时可见。因此，开发者工具（spring-boot-devtools）默认禁用缓存这个功能。

缓存选项通常配置在application.properties文件中，比如Thymeleaf提供了spring.thymeleaf.cache属性，相比于手动设置这些属性，spring-boot-devtools模块会自动应用敏感的development-time配置。

查看DevToolsPropertyDefaultsPostProcessor获取完整的被应用的属性列表。

##20.2 自动重启

使用spring-boot-devtools的应用，只要类路径（classpath）下的文件有变动，就会自动重启。这是一个很有用的功能当使用IDE开发时，因为可以很快得到代码改变的反馈。默认情况下，类路径（classpath）下任何指向文件夹的实体都会被监控，注意有一些资源的修改比如静态assets，视图模板是[不需要重启应用](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-devtools)。

```
* **触发重启** 由于DevTools监控类路径（classpath）下的资源，所以唯一触发重启的方式就是更新类路径（classpath）。引起类路径更新的方式取决于你使用的IDE，在Eclipse里，保存一个修改的文件将引起类路径更新，并触发重启。在IntelliJ IDEA中，构建工程`（Build → Make Project）`有同样效果。

```
* 你也可以通过支持的构建工具（比如，Maven和Gradle）启动应用，只要开启fork功能，因为DevTools需要一个隔离的应用类加载器执行正确的操作。Gradle和Maven默认支持该行为，只要把DevTools加在类路径中。


* 自动重启跟LiveReload可以一起很好的工作，具体参考20.3章节。如果你使用JRebel，自动重启将禁用以支持动态类加载，其他devtools特性，比如LiveReload，属性覆盖仍旧可以使用。

* DevTools依赖应用上下文的shutdown钩子来关闭处于重启过程的应用，如果禁用shutdown钩子`（SpringApplication.setRegisterShutdownHook(false)）`，它将不能正常工作。

* 当判定classpath下实体的改变是否会触发重启时，DevTools自动忽略以下工程：spring-boot，spring-boot-devtools，spring-boot-autoconfigure，spring-boot-actuator和spring-boot-starter。

```
[RestartVsReload] **Restart vs Reload** Spring Boot提供的重启技术是通过使用两个类加载器实现的。没有变化的类（比如那些第三方jars）会加载进一个基础（basic）classloader，正在开发的类会加载进一个重启（restart）classloader。当应用重启时，restart类加载器会被丢弃，并创建一个新的。这种方式意味着应用重启通常比冷启动（cold starts）快很多，因为基础类加载器已经生成并且可用。

如果发现重启对于你的应用来说不够快，或遇到类加载的问题，那你可以考虑重载技术，比如JRebel，这些技术是通过重写它们加载过的类实现的。Spring Loaded提供了另一种选择，然而很多框架不支持它，也得不到商业支持。

```

###20.2.1 例外资源

某些资源的变化没必要触发重启，比如Thymeleaf模板可以随时编辑。默认情况下，位于`/META-INF/maven，/META-INF/resources，/resources，/static，/public`或`/templates`下的资源变更不会触发重启，但会触发一个实时加载（live reload）。如果你想自定义例外资源，你可以使用`spring.devtools.restart.exclude`属性，比如，如果仅仅不要`/static`和`/public`，你可以这样设置：

```
spring.devtools.restart.exclude=static/**,public/**

```
* 如果你想保留默认属性，并添加额外的例外规则，可以使用`spring.devtools.restart.additional-exclude`属性作为代替。

###20.2.2 查看其他路径

你也许想让你的应用在改变没有位于classpath下的文件时也会重启或重新加载。为了实现这个，你可以使用`spring.devtools.restart.additional-paths`属性来配置监控变化的额外路径。你可以使用[上面描述](###20.2.1 例外资源)过的`spring.devtools.restart.exclude`属性去控制额外路径下的变化是否触发一个完整重启或只是一个实时[重新加载]()。

###20.2.3 禁用重启

如果不想使用重启特性，你可以通过`spring.devtools.restart.enabled`属性来禁用它，通常情况下可以在`application.properties`文件中设置（依旧会初始化重启类加载器，但它不会监控文件变化）。

如果需要彻底禁用重启支持，比如，不能跟某个特殊库一块工作，你需要在调用`SpringApplication.run(…​)`之前设置一个系统属性，如下：

```
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}

```
###20.2.4 使用触发器文件

如果使用一个IDE连续不断地编译变化的文件，你可能倾向于只在特定时间触发重启，触发器文件可以帮你实现该功能。触发器文件是一个特殊的文件，只有修改它才能实际触发一个重启检测。改变该文件只会触发检测，实际的重启只会在Devtools发现它必须这样做的时候，触发器文件可以手动更新，也可以通过IDE插件更新。

使用`spring.devtools.restart.trigger-file`属性可以指定触发器文件。

* 你可能想将`spring.devtools.restart.trigger-file`属性设置为全局设置，这样所有的工程表现都会相同。


###20.2.5 自定义restart类加载器

正如以上[Restart vs Reload](RestartVsReload)章节讨论的，重启功能是通过两个类加载器实现的。对于大部分应用来说是没问题的，但有时候它可能导致类加载问题。

默认情况，在IDE里打开的项目会通过'restart'类加载器加载，其他常规的.jar文件会使用'basic'类加载器加载。如果你工作在一个多模块的项目下，并且不是每个模块都导入IDE里，你可能需要自定义一些东西。你需要创建一个META-INF/spring-devtools.properties文件，spring-devtools.properties文件可以包含restart.exclude.，restart.include.前缀的属性。include元素定义了那些需要加载进'restart'类加载器中的实体，exclude元素定义了那些需要加载进'basic'类加载器中的实体，这些属性的值是一个将应用到classpath的正则表达式。

例如：
```
restart.include.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar

```
* 所有属性的keys必须唯一，只要以`restart.include.`或`restart.exclude.`开头都会考虑进去。

* 所有来自类路径的`META-INF/spring-devtools.properties`都会被加载，你可以将文件打包进工程或工程使用的库里。

###20.2.6 已知限制

重启功能不能跟使用标准`ObjectInputStream`反序列化的对象工作，如果需要反序列化数据，你可能需要使用Spring的`ConfigurableObjectInputStream`，并结合`Thread.currentThread().getContextClassLoader()`。

不幸的是，一些第三方库反序列化时没有考虑上下文类加载器，如果发现这样的问题，你需要请求原作者给处理下
