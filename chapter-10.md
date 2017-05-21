## 第10章 安装Spring Boot

Spring Boot 可以和常见的Java开发工具一同使用，也可以从Spring Boot CLI（SDKMAN）直接安装。不管怎样，你必须提前安装Java SDK 1.8或更高本版。 如果你已经安装了Java， 你可以使用以下命令查看当前的Java版本：

```
$ java -version
```

如果你是Java开发的新手，或者你只是尝试一下Spring Boot。你可能会想先试试Spring Boot CLI。或者阅读一下常用安装说明。

## 10.1 Java 开发者的安装说明

你可以像使用任何一个标准Java库一样使用Spring Boot。只需简单地在你的classpath中添加相应的`spring-boot-*.jar`。

Spring Boot不需要什么特殊的集成工具。 你可以用任何IDE或者文本编辑器来编写和调式， Spring Boot并没有什么特别的，你可以像运行任何其他Java程序一样运行和调试它。

虽然你可以通过复制Spring Boot库文件（来将Spring Boot增加到你的类路径中），我们还是建议你使用支持依赖管理的构建工具（如： Maven 或 Gradle）。

### 10.1.1 Maven安装
Spring Boot和Apache Maven 3.2及以上的版本兼容。如果你没有安装Maven，你可以从maven.apache.org找到相关的指南。


> 在很多操作系统上，可以使用包管理器来安装Maven，如果你使用OSX Homebrew的话，你可以用`brew install maven`来安装。Ubuntu的用户可以通过`sudo apt-get install maven`来安装。

Spring Boot的依赖们都会在`org.springframework.boot`的这个`groupId`下。通常，你的Maven Pom文件会继承`spring-boot-starter-parent`项目，应用一个或者多个“Starter”的依赖。Sprint Boot也提供了一个可选的Maven插件用于创建可执行的jar文件。

这里我们提供一个常见`pom.xml`文件作为参考：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.BUILD-SNAPSHOT</version>
    </parent>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <!-- Add Spring repositories -->
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

> 通过`spring-boot-starter-parent`使用Spring Boot是一个不错的方式，但也不是任何时候都适用。有时候，你可能需要继承其他的父POM文件，或者你就是不喜欢默认的设置。 另一种使用`import`域的方法请参见"13.2.2 使用Spring Boot时不添加父POM"

### 10.1.2 Gradle安装
Spring Boot和Gradle 3（3.4或者以上的版本）兼容。如果你并没有安装Gradle，你可以参考 www.gradle.org上的指南。

Spring Boot的依赖都在`org.springframework.boot`这个`group`下。通常你的项目会应用一个或多个"Starter"依赖。Spring Boot提供了一个很好用的Gradle Plugin，它可以用于便捷的声明引用或生成可执行的jar文件。

```
在你想要构建一个项目的时候，Gradle Wrapper 提供了一个很好的获取Gradle的方式。它是你和你的代码一同提交的用于启动构建过程的一小段脚本和库。
```

这里有一个常见的`build.gradle`文件作为参考：

```
buildscript {
    repositories {
        jcenter()
        maven { url 'http://repo.spring.io/snapshot' }
        maven { url 'http://repo.spring.io/milestone' }
    }
    dependencies {
        classpath 'org.springframework.boot:spring-boot-gradle-plugin:2.0.0.BUILD-SNAPSHOT'
    }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'

jar {
    baseName = 'myproject'
    version =  '0.0.1-SNAPSHOT'
}

repositories {
    jcenter()
    maven { url "http://repo.spring.io/snapshot" }
    maven { url "http://repo.spring.io/milestone" }
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

## 10.2 安装Spring Boot CLI
Spring Boot CLI是一个可以用来快速搭建Spring原型系统的命令行工具。它可以执行Groovy脚本，这意味着你可以使用一种你熟悉的类Java的语法，同时又不用引入大量的样本代码。

### 10.2.1 手动安装
你可以从Spring软件库中下载Spring CLI的安装文件：

* spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.zip
* spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin.tar.gz
最新的测试版本可以在这里找到。

下载完成后，可以根据解压包中INSTALL.txt文件的指南进行安装。简而言之，在`zip`文件的`bin/`文件夹中有一个`spring`脚本文件（如果是windows电脑的话，是`spring.bat`）。或者，你可以用`jar -jar`命令执行`.jar`文件（脚本文件可以帮助你确保类路径是正确的）。

### 10.2.2 使用SDKMAN!安装
SDKMAN!（软件开发工具包管理器）用于管理包括Groovy和Spring Boot CLI在内的具有多个版本的SDK。你可以从sdkman.io下载SDKMAN!并通过

```
$ sdk install springboot
$ spring --version
Spring Boot v2.0.0.BUILD-SNAPSHOT
```

安装Spring Boot。如果你正在开发CLI的新特性并想便捷的访问你刚刚构建的版本，则可以：

```
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-2.0.0.BUILD-SNAPSHOT-bin/spring-2.0.0.BUILD-SNAPSHOT/
$ sdk default springboot dev
$ spring --version
Spring CLI v2.0.0.BUILD-SNAPSHOT
```
这样可以在本地安装一个名为`dev`的`spring`实例。它指向你的构建路径，当每次你重新构建Spring Boot时，`spring`将会更新。
你可以通过这样查看：

```
$ sdk ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 2.0.0.BUILD-SNAPSHOT

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```

### 10.2.3 OSX Homebrew 安装
如果你是一个MAC用户并且使用Homebrew，你可以这样安装Spring Boot CLI

```
$ brew tap pivotal/tap
$ brew install springboot
```
Homebrew将会把`Spring`安装到`/use/local/bin`。
> 如果你找不到这个软件（formula)，可能你的brew是较早的版本，可用`brew update`升级并重试。
### 10.2.4 MacPorts 安装
如你是Mac用户并且使用MacPorts，你可以这样安装Spring Boot CLI

```
$ sudo port install spring-boot-cli
```

### 10.2.5 命令行补齐
Spring Boot CLI包含了在BASH和zsh shell上可以帮助命令行补齐的脚本。你可以在任意shell中`source`这些脚本(也叫`spring`)，或者把它放到你的个人或系统的bash初始化补全库中。在Debian系统中，系统脚本位于`/shell-completion/bash`，所有在那个文件夹中的脚本都会在一个新的shell开启时被执行。如果你通过SDKMAN!安装，可以这样手动执行：

```
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
  grab  help  jar  run  test  version
```

> 如果你通过Homebrew或者MacPorts安装Spring Boot CLI，命令行补齐已经自动在shell注册了。

### 10.2.6 快速创建Spring CLI例子
这里有一个你可以用来测试你的安装的极简web应用。创建一个名为`app.groovy`的文件

```
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```
然后在shell中运行：

```
$ spring run app.groovy
```
> 因为需要下载依赖包，当你第一次运行时会需要较长时间。之后会快很多。

在你常用的web浏览器中打开localhost:8080，你应该看到下面的输出：

```
Hello World!
```

## 10.3 从较早的版本升级Spring Boot
如果你要从Spring Boot较早的版本升级，请参考在项目Wiki上的发布日志。你可以找到升级指南以及每个版本“值得关注的新”功能列表。

要使用包管理工具命令（比如 `brew update`）来升级CLI，或者要手动安装CLI的话，可以参考标准指南，同时请切记更新你的`Path`环境变量，删除之前版本的应用。
