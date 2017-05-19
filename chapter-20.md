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
