# 第14章 调整并组织代码结构

Spring Boot并不需要任何具体的源代码文件结构,然而,遵循最佳实践是能起到一定帮助作用的.

## 14.1 使用"默认"的package

当一个类没有包括任何的`package`声明时,这个类便会在"默认package"中.在大部分情况下是不推荐使用"默认package"的,而且应当尽量避免.使用它还会对Spring Boot应用中的`@ComponentScan`,`@EntityScan`或者`@SpringBootApplication`注释造成一定的问题,它会使每一个Jar中的每一个类都被读取.

>我们推荐你使用Java所推荐的package命名准则,并且使用反向的域名名称(比如说`com.example.project`).

## 14.2 定位应用的main类

我们推荐将应用的main类放在应用的根目录下.`@EnableAutoConfiguration`注释一般会在你的main类上,他会对某些项声明一个起始的"搜索package".比如说,如果你在写一个JPA应用,在有`@EnableAutoConfiguration`注释的类的package会被用来搜索`@Entity`项.

使用根目录也可以允许`@ComponentScan`注释在不需要制定`basePackage`属性的情况下被使用.你也可以使用`@SpringBootApplication`注释如果你的main类是在根目录package里.

这是一个典型的项目文件结构:

```
com
 +- example
     +- myproject
         +- Application.java
         |
         +- domain
         |   +- Customer.java
         |   +- CustomerRepository.java
         |
         +- service
         |   +- CustomerService.java
         |
         +- web
             +- CustomerController.java
```

`Application.java`文件中会声明`main`方法,以及最基本的`@Configuration`注释.

```
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
