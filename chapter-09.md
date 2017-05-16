# 第9章 系统配置要求

Spring Boot 2.0.0 要去Java 8和 Spring Framework 5.0.0 RC1 或更高。明确的构建支持有Maven \(3.2+\)和Gradle 3 \(3.4+\)。

## 9.1 Servlet容器

下面的嵌入式`servlet`容器支持开箱即用:

| Name | Servlet Version |
| :--- | :--- |
| Tomcat 8.5 | 3.1 |
| Jetty 9.4 | 3.1 |
| Undertow 1.3 | 3.1 |

Spring Boot 程序同时也支持任何 Servlet 3.0+ 兼容的容器

