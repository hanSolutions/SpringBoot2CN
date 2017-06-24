#29. 与SQL数据库联动

Spring构架为SQL数据库联动提供了广泛的支持。从用 `JdbcTemplate` 对JDBC直接访问到完全'对象关系映射 (object relational mapping)'技术，例如Hibernate. Spring Data提供了更上层楼的功能，从接口直接生成 `Repository` 建立方法，还有像惯例一样用你方法名字生成检索式。

##29.1 配置一个数据源

Java的 `javax.sql.DataSource` 接口提供了与数据库联接的一个标准方法。传统上数据源通过 `URL` 和一些凭证(credentials)去建立数据库联接。

####技巧
> 察看['如何'章节](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-configure-a-datasource)寻求更多高级的事例，通常情况下获得配置数据源的全部控制。

###29.1.1 嵌入式数据库的支持

通常开发一个嵌入式内存数据库的程序会很方便。显然，内存数据库并没有提供持久存储；你需要在你的程序开始时填充你的数据库并且在你的程序结束的时候需要准备好清理掉这些数据。

####技巧
> '如何'章节包含了[如何去初始化一个数据库章节](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-database-initialization)

Spring Boot可以自动配置嵌入式[H2](http://www.h2database.com/), [HSQL](http://hsqldb.org/) 和 [Derby](https://db.apache.org/derby/) 数据库。你不需要提供任何联接用URL，只需要在你想要使用的嵌入式数据库中加入相关性。

####注解
> 如果你在测试时使用这个特性，你会注意到你的测试套件使用同样的数据库无论你使用多少应用内容。如果你希望每个内容有不同的嵌入式数据库，你应该设置 `spring.datasource.generate-unique-name` 为`true`。

例如，典型的POM相关将会是：
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
	<groupId>org.hsqldb</groupId>
	<artifactId>hsqldb</artifactId>
	<scope>runtime</scope>
</dependency>
```

####注解
> 嵌入式数据库需要一个在 `spring-jdbc` 上的相关来被自动配置。在这个事例里通过 `spring-boot-starter-data-jpa` 实现。

####技巧
> 如果在某些原因下，你为嵌入式数据库配置了连接URL，需要注意的是数据库的自动关闭功能是禁止的。如果你用的是H2，你需要使用 `DB_CLOSE_ON_EXIT=FALSE` 来实现。如果你用的是HSQLDB, 你需要确认  `shutdown=true` 没有被使用。禁止数据库的自动关闭功能可以允许Spring Boot在数据库关闭的时候去控制， 从而确保在访问不再需要的数据库时会发生。

###29.1.2 与生产数据库连接

生产数据库的连接也可以用 DataSource 自动配置。这里列举了选择特定方法的逻辑：
* 我们倾向于HikariCP 因为它的表现和并发性，所以如果它可用我们总是选择HikariCP。
* 除此之外， 如果Tomcat的 `DataSource` 池可用我们会用。
* 如果 HikariCP 和 Tomcat 的数据源池都不可用，但是如果 Commons DBCP2 可用，我们会用。

如果你用了 `spring-boot-starter-jdbc` 或者 `spring-boot-starter-data-jpa` 'starters', 你会自动地获得 `HikariCP` 相关。

####注解
> 你可以完全省略上面的逻辑并且指定连接池使用 `spring.datasource.type` 特性。这点在你的程序运行在Tomcat容器内，并且`tomcat-jdbc` 是默认提供时十分重要。

####技巧
> 我们时刻可以人工的配置额外的连接池。如果你定义你自己的 `DataSource` bean???， 自动配置就不会发生。

数据源构造被外在构造属性控制，这个属性在 `spring.datasource.*` 。举个例子，你或许可以在 `application.properties` 里声明下面的部分：
```
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource,password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

####注解
> 你应该至少指定URL使用 `spring.datasource.url` 属性或者Spring Boot将会尝试自动配置一个嵌入式数据库。

####技巧
> 你不用常常规定 `driver-class-name` 因为 Spring Boot 可以从大部分数据库 `url` 中检测出来。

####注解
> 为了让 `DataSource` 池能够被创造我们需要能够效验一个有效的 `Driver` 类是可用的，因此我们在做任何事之前要先检测它。I.e. 如果你设定 `spring.datasource.driver-class-name=com.mysql.jdbc.Driver` 那么那个类别必须是可加载的。

请看 [`DataSourceProperties`](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java)　来获得更多的支持选项。这些标准选项始终会实现无论实际的安装配置。用它们各自前缀来精确调整实行特定的设置是可能的 （`spring.datasource.hikari.*`, `spring.datasource.tomcat.*`, 和 `spring.datasource.dbcp2.*`）。请参考你正在使用的连接实行方法文档来获得更多细节。

例如，如果你正在使用 [Tomcat连接池](https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html#Common_Attributes) 你可以定制很多额外的设置：
```java
#Number of ms to wait before throwing an exception if no connection is available.
spring.datasource.tomcat.max-wait=10000

#Maximum number of active connections that can be allocated from this pool at the same time.
spring.datasource.tomcat.max-active=50

#Validate the connection before borrowing it from the pool.
spring.datasource.tomcat.test-on-borrow=true
```
###29.1.3 JNDI 数据源的连接
如果你部属你的Spring Boot程序到一个程序服务器，你可能希望用你程序服务器内建的特性配置还有管理你的数据源并且用JNDI访问它。

`spring.datasource.jndi-name` 特性可以用来作为从一个特定的JNDI地点访问 `DataSource` 的 `spring.datasource.url`, `spring.datasource.username` 和 `spring.datasource.password` 特性的替代品。举个例子，下面在 `application.properties` 的代码展示了你怎样访问一个被定义为 `DataSource` 的 JBoss:
```java
spring.datasource,jndi-name=java:jboss/datasources/customers
```

##29.2 使用 JdbcTemplate

Spring的 `JdbcTemplate` 和 `NamedParameterJdbcTemplate` 类是自动配置的，你可以在你自己的代码当中直接 `@Autowire` 它们：
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // ...

}
```

你可以用 `spring.jdbc.template.*` 特性来自定义一些模版属性：
```java
spring.jdbc.template.max-rows=500
```

####技巧
> `NamedParameterJdbcTemplate` 在后台重新使用了同样的 `JdbcTemplate` 实例。如果不止一个 `JdbcTemplate` 被定义了并且没有主要候选者存在，`NamedParameterJdbcTemplate` 是不会自动配置的。

##29.3 JPA 和 'Spring 数据'

Java Persistence API 是个标准的技术，可以让你'映射'对象到关系数据库。`spring-boot-starter-data-jpa` POM提供了一个快速的开始方式。它提供了一下几个重要的相关性：
* Hibernate - 最重要的JPA实行方法之一。
* Spring Data JPA - 使实行基于JPA的知识库变得方便。
* Spring ORMs - 核心ORM得到Spring构架的支持。

####技巧
> 我们在这里不会解释太多关于JPA或者Spring Data细节。你可以跟随网站 [spring.io](https://spring.io/) 的['用JPA访问数据'](https://spring.io/guides/gs/accessing-data-jpa/)指南以及阅读 [Spring Data JPA](http://projects.spring.io/spring-data-jpa/) 还有 [Hibernate](http://hibernate.org/orm/documentation/) 参考资料。

###29.3.1 实体类别

传统来说，JPA '实体'类别是在 `persistence.xml` 文件里规定的。在Spring Boot里这个文件不是必须的而使用替代者'Entity Scanning'。默认情况下，所有在你主要配置类别下面的组件（拥有注释 `@EnableAutoConfiguration` 或者 `@SpringBootApplication`）会被搜寻。

任何含有注释 `@Entity`, `@Embeddedable` 或者 `@MappedSuperclass` 的类别会被研究。一个典型的实体类别应该跟下面的例子类似：
```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String state;

    // ... additional members, often include @OneToMany mappings

    protected City() {
        // no-args constructor required by JPA spec
        // this one is protected since it shouldn't be used directly
    }

    public City(String name, String state) {
        this.name = name;
        this.country = country;
    }

    public String getName() {
        return this.name;
    }

    public String getState() {
        return this.state;
    }

    // ... etc

}
```

####技巧
> 你可以用 `@EntityScan` 注解来定制实体扫描机制的地点。请看 [章节76.4 ”从Spring配置里分离 @Entity 定义“](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-separate-entity-definitions-from-spring-configuration) 的 如何。

###29.3.2 Spring Data JPA 库

Spring Data JPA 库是一种你可以定义访问数据的借口。JPA询问会以你的方法名字自动创造。举个例子，一个 `CityRespository` 借口可以宣称一个 `findAllByState(String state)` 方法来找到一个给定州的所有城市。

对于更复杂的询问，你可以用Spring Data [`Query`](http://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/Query.html) 注解注释你的方法。

Spring Data 库通常从 [`Repository`](http://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/Repository.html) 或者 [`CrudRepository`](http://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) 借口得到延伸。如果你使用自动配置，知识库会在包含你主要配置类别的组件里搜寻（拥有注解 `@EnableAutoConfiguration` 或者 `@SpringBootApplication`）。

这里是一个典型的Spring Boot库:
```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndCountryAllIgnoringCase(String name, String country);

}
```

####技巧
> 我们仅仅领略了Spring Data JPA的皮毛。请查询[对应的文档](http://projects.spring.io/spring-data-jpa/)获得完整的细节。

###29.3.3 建造和放弃 JPA 数据库

默认情况下，JPA数据库__只有__在你用的是嵌入式数据库（H2, HSQL 或者 会自动Derby）时会自动建造。你可以明确地使用 `spring.jpa.*` 特性来配置JPA设置。举个例子，你可以在 `application.properties` 里加上下面的代码去建造和放弃表格。
```
spring.jpa.hibernate.ddl-auto=create-drop
```

####注解
> Hibernate自己关于这个的内在特性名（如果你愿意记住它更好）为 `hibernate.hbm2ddl.auto` 。你可以用 `spring.jpa.properties.*` 设定它，与其他Hibernate本身特性一起 (前缀在把它们加入实体管理器之前被剥离）。举例：
```
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```
传递 `hibernate.globally_quoted_identifiers` 到Hibernate实体管理器。

默认情况下，DDL执行 （或者验证）会被延期到 `ApplicationContext` 开始。`spring.jpa.generate-ddl` 标记同样存在，但是它在Hibernate自动配置启动的情况下是不会被使用的，因为 `ddl-auto` 的设置是更细节的。

###29.3.4 开启 EntityManager 视图模式

如果你在运行一个网络程序，Spring Boot会默认注册 [`OpenEntityManagerInViewInterceptor`](http://docs.spring.io/spring/docs/5.0.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html) 去运用”Open EntityManager in View"模式。i.e. 允许在网络视图下的惰性载入。如果你不希望这样的行为，你应该在 `application.properties` 里设置 `spring.jpa.open-in-view` 为 `false`。

##29.4 使用 H2 的网络控制台

[H2数据库](http://www.h2database.com/)提供了一个Spring Boot可以自动为你配置的[基于浏览器的控制台](http://www.h2database.com/html/quickstart.html#h2_console)。这个网络平台会在下列条件达成时自动配置：
* 你正在开发一个网络程序
* `com.h2database:h2` 是在 classpath
* 你正在使用[Spring Boot的开发者工具](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-devtools)

####技巧
> 如果你没有使用Spring Boot的开发者工具，但是还是希望使用H2的控制台，你可以设置 `spring.h2.console.enabled` 特性为 `true`。H2控制台只可以在开发环境下使用所以一定要注意在成平时 `spring.h2.console.enabled` 没有被设置成 `true`。

###29.4.1 改变H2控制台的路径

默认情况下控制台在 `/h2-console`。你可以用 `spring.h2.console.path` 特性自定义控制台的路径。

###29.4.2 保护H2控制台

当Spring保护在classpath并且基本认证是开启的，H2控制台会被基本认证自动保护起来。下列特性可以用来自定义安全配置：
* `security.user.role`
* `security.basic.authorize-mode`
* `security.basic.enabled`

##29.5 使用jOOQ

Java对象导向查询 ([jOOQ](http://www.jooq.org/))是可以从你数据库中生成Java代码并且让你使用它自己熟练的API建立类型安全的SQL询问式的一个来自 [Data Geekery](http://www.datageekery.com/) 流行产品。它的商业或者开源版本都可以被Spring Boot使用。

###29.5.1 代码生成

为了使用jOOQ类型安全询问式，你需要去你数据库概要生成Java的类。你可以根据[jOOQ的用户手册](http://www.jooq.org/doc/3.6/manual-single-page/#jooq-in-7-steps-step3)步骤。如果你在使用 `jooq-codegen-maven` 插件（并且你还使用`spring-boot-starter-parent` "parent POM"）你可以安全地删掉插件的`<version>` 标签。你还可以使用Spring Boot定义版本的变量（e.g. `h2.version`）去声明插件的数据库相关性。这里是一个例子：
```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <executions>
        ...
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency>
    </dependencies>
    <configuration>
        <jdbc>
            <driver>org.h2.Driver</driver>
            <url>jdbc:h2:~/yourdatabase</url>
        </jdbc>
        <generator>
            ...
        </generator>
    </configuration>
</plugin>
```

###29.5.2 使用DSLContext

jOOQ提供的熟练API可以通过 `org.jooq.DSLContext` 借口初始化。Spring Boot会自动配置一个 `DSLContext` 作为一个Spring Bean并且把它和你程序的 `DataSource` 链接起来。想要使用 `DSLContext` 你可以用 `@Autowire` ：
```java
@Component
public class JooqExample implements CommandLineRunner {

    private final DSLContext create;

    @Autowired
    public JooqExample(DSLContext dslContext) {
        this.create = dslContext;
    }

}
```

####技巧
> jOOQ手册倾向使用 `create` 变量名称去命名 `DSLContext` ,我们在这个例子里使用了相同的方法。

接下来你可以使用 `DSLContext` 去组成你的询问式：
```java
public List<GregorianCalendar> authorsBornAfter1980() {
    return this.create.selectFrom(AUTHOR)
        .where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
        .fetch(AUTHOR.DATE_OF_BIRTH);
}
```

###29.5.3 jOOQ SQL 通俗语

Spring Boot确定SQL通俗语为你的数据源使用，除非 `spring.jooq.sql-dialect` 属性被设置了。如果通俗语不能被检测到，`DEFAULT` 被使用。

####注解
> Spring Boot只能自动配置被开源版本jOOQ支持的通俗语。

###29.5.4 自定义jOOQ

通过定义你自己 `@Bean`更高级的自定义可以实现，这个定义会在jOOQ `Configuration` 创建时使用到。你可以定义beans为下列jOOQ类型：
* `ConnectionProvider`
* `TransactionProvider`
* `RecordMapperProvider`
* `RecordListenerProvider`
* `ExecuteListenerProvider`
* `VisitListenerProvider`

你还可以创建你自己的 `org.jooq.Configuration``@Bean` 如果你想完全掌控jOOQ的配置。
