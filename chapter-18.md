@SpringBootApplication注释使用

很多Spring Boot开发者习惯在主类上方注释@Configuration，@EnableAutoConfiguration，及@ComponentScan。因为这些注释会经常出现在一起使用（特别是你按照之前章节所讲述的最佳实践去编译代码），于是Spring Boot为开发者提供了一个便利的注释方法@SpringBootApplication。 

@SpringBootAnnotation注释与@Configuration，@EnableAutoConfiguration，及@ComponentScan三者默认的属性等价

```java
package com.example.myproject;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
