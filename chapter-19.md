##第十九章：应用程序执行

将你的应用程序打包成jar文件及使用嵌入式HTTP服务器的一个最大优点是执行方式和执行其他应用程序大体相同。调试Spring Boot应用程序也非常简易，无需添加额外的IDE插件和附加物。

本章节只针对于jar文件的打包方法。如果你想将你的应用程序打包成war文件，请参考你使用的服务器及IDE的说明书。

###第一小节：使用IDE执行应用程序

在IDE里，Spring Boot应用程序可以以简易Java应用程序来执行。但是，你需要先导入程序项目。不同IDE和搭建系统的导入方式存异。大部分IDE可以直接导入Maven项目，例如Eclipse使用者可以在文件下拉菜单里选择导入 - 已有的Maven项目。

假如你无法直接导入项目至IDE，你可以使用搭建插件来生成IDE的附加数据。Maven有Eclipse和IDEA的插件，Gradle对于不同的IDE也有不同的插件。

假如你无意间执行网页应用程序两次，你会看到一个错误提示"Port already in use"（接口已被使用）。STS使用者与其用Run键，可使用Relaunch键，这样可以确保关闭已经打开的实例。

###第二小节：执行已打成包的应用程序

假如你使用Spring Boot Maven或Gradle插件去创建一个可执行的jar文件，你可以使用```java -jar```。例如：

```js
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar
```

你也可以打开远程调试功能来执行已打包好的应用程序，这样就会附加一个调试器在应用程序上。

```js
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myproject-0.0.1-SNAPSHOT.jar
```

###第三小节：Maven插件的使用

Spring Boot Maven插件包含一个```run```的目标可以用来编译及执行你的应用程序。程序的执行就像在IDE里去执行压缩前的项目文件一样。

```js
$ mvn spring-boot:run
```

你可能也会用到一些有用的系统环境变量如下：

```js
$ export MAVEN_OPTS=-Xmx1024m
```

###第四小节：Gradle插件的使用

Spring Boot Gradle插件另外还包含一个```bootRun```的任务来已未压缩的形式去执行应用程序。你每一次运用```org.springframework.boot```和```java```插件时，```bootRun```任务都会被加进Gradle插件里。

```js
$ gradle bootRun
```

你可能也会用到一个很有用的操作系统环境变量：

```js
$ export JAVA_OPTS=-Xmx1024m
```

###第五小节：热插拔

因为Spring Boot应用程序是一个简易Java应用程序，JVM热插拨应该可以直接使用。JVM热插拨在替换字节码上有局限性，可以使用JRebel来替换。```spring-boot-devtools```组件还包含快速重启程序的功能。

详情请参考第二十张"开发者的工具"和第八十三章"怎样使用热插拨"