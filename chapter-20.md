#第20章 开发者工具

Spring Boot包含了一些额外的工具集，用于提升体验Spring Boot应用的开发乐趣。`spring-boot-devtools`模块可以被包含到任何模块中来提供开发时的特性，你只需简单的将该模块的依赖添加到你的构建中：

**Maven.**
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```
**Gradle.**
```properties
dependencies {
    compile("org.springframework.boot:spring-boot-devtools")
}
```

![leaf_icon](icons\leaf_icon.png) 当执行一个完整的，打包过的应用时，开发者工具（devtools）会被自动禁用。如果你的应用是使用`java -jar`或特殊的类加载器启动，都会被认为是一个产品级的应用（production application），从而禁用开发者工具。设置该依赖级别为optional从而防止devtools传递到项目中的其他模块是个好经验（Best Practice）。由于Gradle不支持`optional`依赖，所以你可能同时要了解下[propdeps-plugin](https://github.com/spring-projects/gradle-plugins/tree/master/propdeps-plugin)。

![star_icon](icons\star_icon.png) 由于再打包的压缩应用文件（repackaged archives）默认不包含开发者工具。如果想使用开发者工具的[一些远程特性](##20.5 远程应用),你需要禁用`excludeDevtools`构建属性来添加它进应用，Maven和Gradle都支持该属性。


##20.1 默认属性

Spring Boot支持的一些库（libraries）使用缓存提高性能，比如，[模板引擎（template engines）](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-spring-mvc-template-engines)会缓存已经编译的模版来避免反复地解析模版文件。当要提供静态资源（文件）时，Spring MVC还能够添加HTTP缓存头字段

虽然缓存会给实际产品带来很大的好处，但它会降低开发效率在产品的开发阶段。它会妨碍你对你的程序修改结果及时可见。因此，开发者工具（spring-boot-devtools）默认禁用缓存这个功能。

缓存选项通常配置在`application.properties`文件中，比如Thymeleaf提供了`spring.thymeleaf.cache`属性，相比于手动设置这些属性，`spring-boot-devtools`模块会自动应用敏感的开发时的配置。

![star_icon](icons\star_icon.png) 查看DevToolsPropertyDefaultsPostProcessor获取完整的被应用的属性列表。

##20.2 自动重启

使用spring-boot-devtools的应用，只要类路径（classpath）下的文件有变动，就会自动重启。这是一个很有用的功能当使用IDE开发时，因为可以很快得到代码改变的反馈。默认情况下，类路径（classpath）下任何指向文件夹的实体都会被监控，注意有一些资源的修改比如静态assets，视图模板是[不需要重启应用](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-devtools)。


>**触发重启**
由于DevTools监控类路径（classpath）下的资源，所以唯一触发重启的方式就是更新类路径（classpath）。引起类路径更新的方式取决于你使用的IDE，在Eclipse里，保存一个修改的文件将更新类路径，并触发重启。在IntelliJ IDEA中，构建工程`（Build → Make Project）`有同样效果。


![leaf_icon](icons\leaf_icon.png) 你也可以通过支持的构建工具（比如，Maven和Gradle）启动应用，只要开启fork功能，因为DevTools需要一个隔离的应用类加载器执行正确的操作。Gradle和Maven默认支持该行为，只要把DevTools加在类路径中。


![star_icon](icons\star_icon.png) 自动重启跟LiveReload可以一起很好的工作，具体参考20.3章节。如果你使用JRebel，自动重启将禁用以支持动态类加载，其他devtools特性，比如LiveReload，属性覆盖仍旧可以使用。

![leaf_icon](icons\leaf_icon.png) DevTools依赖应用上下文的shutdown钩子来关闭处于重启过程的应用，如果禁用shutdown钩子`（SpringApplication.setRegisterShutdownHook(false)）`，它将不能正常工作。

![leaf_icon](icons\leaf_icon.png) 当判定classpath下实体的改变是否会触发重启时，DevTools自动忽略以下工程：spring-boot，spring-boot-devtools，spring-boot-autoconfigure，spring-boot-actuator和spring-boot-starter。


>**Restart vs Reload**  Spring Boot提供的重启技术是通过使用两个类加载器实现的。没有变化的类（比如那些第三方jars）会加载进一个基础（basic）classloader，正在开发的类会加载进一个重启（restart）classloader。当应用重启时，restart类加载器会被丢弃，并创建一个新的。这种方式意味着应用重启通常比冷启动（cold starts）快很多，因为基础类加载器已经生成并且可用。
>如果发现重启对于你的应用来说不够快，或遇到类加载的问题，那你可以考虑重载技术，比如JRebel，这些技术是通过重写它们加载过的类实现的。



###20.2.1 例外资源

某些资源的变化没必要触发重启，比如Thymeleaf模板可以随时编辑。默认情况下，位于`/META-INF/maven，/META-INF/resources，/resources，/static，/public`或`/templates`下的资源变更不会触发重启，但会触发一个实时加载（live reload）。如果你想自定义例外资源，你可以使用`spring.devtools.restart.exclude`属性，比如，如果仅仅不要`/static`和`/public`，你可以这样设置：

```
spring.devtools.restart.exclude=static/**,public/**
```
* 如果你想保留默认属性，并添加额外的例外规则，可以使用`spring.devtools.restart.additional-exclude`属性作为代替。

###20.2.2 查看其他路径

你也许想让你的应用在改变没有位于classpath下的文件时也会重启或重新加载。为了实现这个，你可以使用`spring.devtools.restart.additional-paths`属性来配置监控变化的额外路径。你可以使用[上面描述](###20.2.1-例外资源)过的`spring.devtools.restart.exclude`属性去控制额外路径下的变化是否触发一个完整重启或只是一个实时[重新加载](##20.3-LiveReload)。

###20.2.3 禁用重启

如果不想使用重启特性，你可以通过`spring.devtools.restart.enabled`属性来禁用它，通常情况下可以在`application.properties`文件中设置（依旧会初始化重启类加载器，但它不会监控文件变化）。

如果需要彻底禁用重启支持，比如，由于不能跟某个特殊库一块工作，你需要在调用`SpringApplication.run(…​)`之前设置一个系统属性，如下：

```
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```
###20.2.4 使用触发器文件

如果使用一个IDE连续不断地编译变化的文件，你可能倾向于只在特定时间触发重启，触发器文件可以帮你实现该功能。触发器文件是一个特殊的文件，只有修改它才能实际触发一个重启检测。改变该文件只会触发检测，实际的重启只会在Devtools发现它必须这样做的时候，触发器文件可以手动更新，也可以通过IDE插件更新。

使用`spring.devtools.restart.trigger-file`属性可以指定触发器文件。

![star_icon](icons\star_icon.png)  你可能想将`spring.devtools.restart.trigger-file`属性设置为全局设置，这样所有的工程表现都会相同。


###20.2.5 自定义restart类加载器

正如以上[Restart vs Reload](RestartVsReload)章节讨论的，重启功能是通过两个类加载器实现的。对于大部分应用来说是没问题的，但有时候它可能导致类加载问题。

默认情况，在IDE里打开的项目会通过'restart'类加载器加载，其他常规的.jar文件会使用'basic'类加载器加载。如果你工作在一个多模块的项目下，并且不是每个模块都导入IDE里，你可能需要自定义一些东西。你需要创建一个`META-INF/spring-devtools.properties`文件.
`spring-devtools.properties`文件可以包含`restart.exclude.，restart.include.`前缀的属性。include元素定义了那些需要加载进'restart'类加载器中的实体，exclude元素定义了那些需要加载进'basic'类加载器中的实体，这些属性的值是一个用来检测classpath的正则表达式。

例如：
```
restart.include.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```
![leaf_icon](icons\leaf_icon.png) 所有属性的keys必须唯一，只要以`restart.include.`或`restart.exclude.`开头都会考虑进去。

![star_icon](icons\star_icon.png)  所有来自类路径的`META-INF/spring-devtools.properties`都会被加载，你可以将文件打包进工程或工程使用的库里。

###20.2.6 已知限制

重启功能不能跟使用标准`ObjectInputStream`反序列化的对象工作，如果需要反序列化数据，你可能需要使用Spring的`ConfigurableObjectInputStream`，并结合`Thread.currentThread().getContextClassLoader()`。

不幸的是，一些第三方库反序列化时没有考虑上下文类加载器，如果发现这样的问题，你需要请求原作者给处理下

##20.3 LiveReload(重新加载)

`spring-boot-devtools`模块包含一个内嵌的LiveReload服务器，它可以在资源改变时触发浏览器刷新。LiveReload浏览器扩展可以免费从[livereload.com](http://livereload.com/extensions/)站点获取，支持Chrome，Firefox，Safari等浏览器。

如果不想在运行应用时启动LiveReload服务器，你可以将`spring.devtools.livereload.enabled`属性设置为`false`。

* 你只能在同一时间运行一个LiveReload服务器，在启动你的应用之前，确保没有别的LiveReload服务器在运行。如果你在IDE中启动多个应用，只有第一个能够获得动态加载功能。

##20.4 全局设置

在`$HOME`文件夹下添加一个`.spring-boot-devtools.properties`的文件可以用来配置全局的devtools设置（注意文件名以"."开头），添加进该文件的任何属性都会应用到你机器上使用该devtools的 **所有** Spring Boot应用。例如，想使用触发器文件进行重启，可以添加如下配置：

**~/.spring-boot-devtools.properties.**

```
spring.devtools.reload.trigger-file=.reloadtrigger
```
##20.5 远程应用

Spring Boot开发者工具并不仅限于本地开发，在运行远程应用时你也可以使用一些功能。远程支持是可选的，你需要确保把devtools包括在再打包的压缩文件包中。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后通过设置`spring.devtools.remote.secret`属性可以启用它，例如：

```
spring.devtools.remote.secret=mysecret
```
![exclamatory_icon_icon](icons\exclamatory_icon.png) 在远程应用上启用spring-boot-devtools有一定的安全风险，生产环境中最好不要使用。

远程devtools支持分两部分：一个是接收连接的服务端端点，另一个是运行在IDE里的客户端应用。如果设置`spring.devtools.remote.secret`属性，服务端组件会自动启用，客户端组件必须手动启动。

###20.5.1 运行远程客户端应用

远程客户端应用程序（remote client application）需要在IDE中运行，你需要使用跟将要连接的远程应用相同的classpath运行`org.springframework.boot.devtools.RemoteSpringApplication`，传参为你要连接的远程应用URL。例如，你正在使用Eclipse或STS，并有一个部署到Cloud Foundry的my-app工程，远程连接该应用需要做以下操作：

* 从Run菜单选择`Run Configurations…`。
* 创建一个新的`Java Application`启动配置（launch configuration）。
* 浏览`my-app`工程。
* 将`org.springframework.boot.devtools.RemoteSpringApplication`作为main类。
* 将`https://myapp.cfapps.io作为参数传递给RemoteSpringApplication`（或其他任何远程URL）。
运行中的远程客户端看起来如下：

```
 .   ____          _                                              __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
 =========|_|==============|___/===================================/_/_/_/
 :: Spring Boot Remote :: 1.4.1.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```
![leaf_icon](icons\leaf_icon.png) 因为远程客户端使用的classpath跟真实应用相同，所以它能直接读取应用配置，这就是`spring.devtools.remote.secret`如何被读取和传递给服务器做验证的。

![star_icon](icons\star_icon.png)  强烈建议使用`https://`作为连接协议，这样传输通道是加密的，密码也不会被截获。

如果需要使用代理连接远程应用，你需要配置`spring.devtools.remote.proxy.host`和`spring.devtools.remote.proxy.port`属性。


###20.5.2 远程更新

远程客户端将监听应用的classpath变化，任何更新的资源都会发布到远程应用，并触发重启，这在你使用云服务迭代某个特性时非常有用。通常远程更新和重启比完整rebuild和deploy快多了。

![leaf_icon](icons\leaf_icon.png) 文件只有在远程客户端运行时才监控。如果你在启动远程客户端之前改变一个文件，它是不会被发布到远程服务器的。

###20.5.3 远程调试通道

Java的远程调试在诊断远程应用问题时很有用，不幸的是，当应用部署在你的数据中心外时，它并不总能够启用远程调试。如果你使用基于容器的技术，比如Docker，远程调试设置起来非常麻烦。

为了绕过这些限制，devtools支持基于HTTP的远程调试通道。远程客户端在`8000`端口提供一个本地服务器，这样远程debugger就可以连接了。一旦连接建立，调试信息就通过HTTP发送到远程应用。你可以使用`spring.devtools.remote.debug.local-port`属性设置不同的端口。

你需要确保远程应用启动时开启了远程调试功能，通常，这可以通过配置`JAVA_OPTS`实现，例如，对于Cloud Foundry，你可以将以下内容添加到`manifest.yml`：

```
---
  env:
    JAVA_OPTS: "-Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n"
```
![star_icon](icons\star_icon.png) 注意你不需要传递一个`address=NNNN`的配置项到`-Xrunjdwp`，如果遗漏了，java会使用一个随机可用端口。

![leaf_icon](icons\leaf_icon.png) 通过Internet来调试远程服务可能会很慢，你可能需要增加IDE的超时时间。例如，在Eclipse中你可以从`Preferences…`选择`Java -> Debug`，改变`Debugger timeout (ms)`为更合适的值（`60000`在多数情况下就能解决）。

![exclamatory_icon_icon](icons\exclamatory_icon.png) 当在IntelliJ IDE中使用远程调试通道时。所有的断电应该设置成暂停线程而不是暂停VM。默认下，IntelliJ会暂停整个VM而不是只暂停触发到断点的进程。这有一个不好的副作用，当暂停管理远程调试通道的进程时，你的debug会话期（session）会停止。详情请参考[IDEA-165769].(https://youtrack.jetbrains.com/issue/IDEA-165769)
