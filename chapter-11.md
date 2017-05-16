# 第11章  开发你的第一个Spring Boot应用

我们首先用Java来开发一个"Hello World!"的网络应用并在其中用到一些Spring Boot的主要功能.我们会用大部分集成编译环境都支持的Maven作为编译工具.

>[spring.io](https://spring.io/)的网站包括很多使用Spring Boot的"入门"教程.如果你想要解决某个具体的问题,请先查看这些示例.你可以到[start.spring.io](https://start.spring.io/)在依赖搜索(dependency seacher)中选择`web`来快速学习接下来的步骤.这样可以自动生成一个新的项目从而[即刻开始编写代码](#11.3 编写代码)

在我们开始之前,打开一个终端窗口并确认你安装了正确版本的Java和Maven.

```
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

```
$ mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T16:41:47+00:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```

>示例项目应该创建在单独的文件夹中.接下来的步骤默认为你已经建立了一个对应的文件夹并且是你的当前目录.

## 11.1 创建POM

我们首先要创建一个`pom.xml`文件.这个`pom.xml`文件是用来搭建整个项目的蓝本.打开文档编辑器并输入以下内容:
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Additional lines to be added here... -->

    <!-- (you don't need this if you are using a .RELEASE version) -->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
            <snapshots><enabled>true</enabled></snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-snapshots</id>
            <url>http://repo.spring.io/snapshot</url>
        </pluginRepository>
        <pluginRepository>
            <id>spring-milestones</id>
            <url>http://repo.spring.io/milestone</url>
        </pluginRepository>
    </pluginRepositories>
</project>
```

这应该可以产生一个可编译的项目,你可以通过`mvn package`来进行测试.(你暂时可以忽略"jar will be empty - no content was marked for inclusion!"的警告信息).

>到这一步,你应该已经可以将项目引入集成开发环境的(大部分现代Java集成开发环境内置了Maven支持).为了简便起见,我们会继续在这个例子中使用纯文本编辑器.

## 11.2 加入类路径依赖

Spring Boot提供了很多"起始包(Starters)"来帮助用户将jar文件加入类路径.我们的示例应用已经在`POM`的`parent`部分使用了`spring-boot-starter-parent`.`spring-boot-start-parent`是一个特殊的起始包,提供并帮助配置了很多Maven的默认值.它还提供了一个`dependency-management`的部分,这一部分可以帮助开发者将很多依赖的库的`版本`进行自动配置，这样开发者便可以不需要再在`pom.xml`声明依赖库的版本.

其他"起始包"会提供某一类具体应用可能用到的所需要的库.因为我们开发的是一个网络应用,我们将加入`spring-boot-starter-web`依赖 -- 但是首先，我们先看一下我们项目目前的情况.

```
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

`mvn dependency:tree`命令会打印出一个项目的树形依赖图.你可以看到`spring-boot-starter-parent`本身并没有任何的依赖库.编辑`pom.xml`并且在`parent`部分的下方加入对`spring-boot-starter-web`的依赖项.

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

如果再次运行`mvn dependency:tree`,你会看到现在比刚才多了很多的依赖库，包括Tomcat网络服务器和Spring Boot.

## 11.3 编写代码

为了完成我们的应用,我们需要创建一个Java文件.在默认情况下,Maven会将`src/main/java`目录中的源代码进行编译，所以你需要遵循这个文件夹结构,然后创建一个文件并命名为`src/main/java/Example.java`:

```
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.stereotype.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Example.class, args);
    }

}
```

即使并没有太多的代码,这里却有很多的知识点,让我们一步一步的来分析一下各个重点部分.

### 11.3.1 关于@RestController和@RequestMapping注释

在`Example` 类中的第一个注释是`@RestController`.这是一个已知的复合型的注释.它给读代码的人是一个提示,而且对于Spring,这个类也具有一个很具体的用处.在我们这个示例中,这一个类是一个web `@Controller`,所以Spring便会在处理传入的Web请求时使用到这个类.

`@RequestMapping`注释提供了"路由"信息.它会告知Spring对于任何HTTP带有路径"/"的请求，Sping应该将请求映射到`home`方法上.这个`@RestController`注释来告知Spring将请求的字符串结果直接返回到请求的呼叫者.

>`@RestController`以及`@RequestMapping`是Spring MVC的注释类(并不是特别针对Spring Boot的).详情请阅读Spring文档中的[MVC部分](http://docs.spring.io/spring/docs/5.0.0.RC1/spring-framework-reference/htmlsingle#mvc)

### 11.3.2 关于@EnableAutoConfiguration注释

第二个类一层的注释是`@EnableAutoConfiguration`.这个注释告诉Spring Boot去根据你所添加的依赖的类从而最大限度的自动对Spring进行配置.因为`spring-boot-starter-web`在依赖库中加入了Tomcat和Spring MVC,自动配置会以开发网络应用(web application)的模式来对Spring行好配置.

>**起始包以及自动配置**
从设计角度,自动配置和起始包是会在一起很好的工作,但是这两部分并不是绝对的捆绑和依赖关系.你可以自己选择在起始包以外的所需要的依赖库,然后Spring Boot仍然会根据所加入的依赖库尽最大努力自动配置你的应用.

### 11.3.3 main方法

我们应用的最后一部分是`main`方法.这是一个标准的Java程序的入口点.我们的main方法通过运行`run`来委托给Spring Boot的`SpringApplication`类.`SpringApplication`会启动我们的应用,启动Spring从而启动已经自动配置好的Tomcat网络服务器.我们需要将`Example.class`以参数的形式传递给`run`方法,从而使`SpringApplication`知道哪个是主要的Spring组成构件.`args`数组同时也会将任何命令行的参数传递进去.

## 11.4 运行示例程序

到此,我们的应用应该已经可以工作了.因为我们在POM中使用了`spring-boot-starter-parent`,我们会有一个很实用的`run`目标(goal)让我们可以用来启动我们的应用.在应用程序的根目录下输入`mvn spring-boot:run`来启动应用程序.

```
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.0.0.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

如果你打开一个浏览器窗口并在地址栏输入[localhost:8080](localhost:8080),你应该可以看到以下的结果.

```
Hello World!
```

要退出应用程序请使用`ctrl-c`按键组合.

## 11.5 制造一个可执行的Jar文件

在示例的最后,我们来创建一个完全独立并可在生产环境中运行的可执行Jar文件.可执行Jar文件(也叫做"fat jars/胖jar")是一个包含了运行你的应用程序所需要的所有依赖库以及依赖的Jar文件的打包文件.

>**可运行Jar文件及Java**
Java没有提供一个标准化的运行嵌套Jar文件的方式(比如说包含在一个Jar文件里的另一个Jar文件).这样对想要发布自我包含的的应用程序造成了一定的问题.
要解决这个问题,许多开发人员选择使用"uber" jar文件. Uber jar文件指的是将所有依赖的Jar包以及库内的类文件打包到一个集成文件内.这种方法的问题在于,开发者将会很难区分在应用中使用了哪些库.另一个就是如果相同的文件名(但文件内容不同)在多个Jar中被使用时，也会造成问题.
Spring Boot使用了[另一种方式](http://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/#executable-jar)来让开发者进行直接的Jar文件嵌套.

要创造一个可执行的Jar文件我们需要在`pom.xml`文件中添加`spring-boot-maven-plugin`.在`dependencies`部分下面添加下面这些代码:
```
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

>`spring-boot-starter-parent`POM包括一个`<executions>`配置来适配`repackage`目标.如果你没有使用parent POM那么你需要自己来声明这一部分配置.更多详情请参考[这里的插件信息](http://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/maven-plugin//usage.html).

保存你的`pom.xml`文件然后在命令行运行`mvn package`.
```
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.0.0.BUILD-SNAPSHOT:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

如果你检查`target`目录,那么你会发现`myproject-0.0.1-SNAPSHOT.jar`.这个文件应该有10MB左右大小.如果你想看一下文件里面的内容可以用命令`jar tvf`:
```
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

你应该还能在`target`目录下找到一个更小的文件叫做`myproject-0.0.1-SNAPSHOT.jar.original`.这个是在被Spring Boot重新打包前,由Maven编译的原始文件.
要执行这个应用,使用`java -jar`命令:
```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.0.0.BUILD-SNAPSHOT)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```

像之前一样使用组合键`ctrl-c`去退出程序.
