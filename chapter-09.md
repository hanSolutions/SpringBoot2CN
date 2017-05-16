## 第9章 系统配置要求

Spring Boot 2.0.0 要去Java 8和 Spring Framework 5.0.0 RC1 或更高。明确的构建支持有Maven \(3.2+\)和Gradle \(1.12 or 2.x\)，不支持Gradle 3

### 9.1 Servlet容器

下面的嵌入式`servlet`容器支持开箱即用：

| Name | Servlet Version | Java Version |
| :--- | :--- | :--- |
| Tomcat 8 | 3.1 | Java 7+ |
| Tomcat 7 | 3.0 | Java 6+ |
| Jetty 9.3 | 3.1 | Java 8+ |
| Jetty 9.2 | 3.1 | Java 7+ |
| Jetty 8 | 3.0 | Java 6+ |
| Undertow 1.3 | 3.1 | Java 7+ |

你也可以部署Spring Boot应用到任何兼容Servlet 3.0+的容器。

