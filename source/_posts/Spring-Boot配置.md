---
title: Spring Boot配置
date: 2020-08-22 01:34:28
categories:
- [Spring Boot基础]
tags: 
- Spring Boot
- Java
---

# 配置文件

 Spring Boot使用一个全局的配置文件，配置文件名是固定的

```
application.properties
application.yml
```

配置文件的作用：修改Spring Boot自动配置的默认值。

`.yml`是YAML（YAML Ain’t Markup Language）语言的文件，以数据为中心，比json、xml等更适合做配置文件。

<!-- more -->

# YAML语法

## 基本语法

- 使用缩进表示层级关系

- 缩进时不允许使用Tab键，只允许只用空格

- 缩进的空格数目不重要，只要相同层级的元素左对齐即可

- 大小写敏感

## 支持的数据结构

**对象（属性和值）：健值对的集合**

```yml
friends:
	name: tom
	age: 20
```

行内写法：

```yml
friends: {name: tom, age: 20}
```

**数组（List、Set）：一组按次序排列的值**

用`-值`表示数组中的一个元素

```yml
pets:
	- cat
	- pig
	- dog
```

行内写法：

```java
pets: [cat,pig,dog]		
```

**字面量（数字、字符串、布尔）：单个的、不可再分的值**

```
k: v：字面直接写

“”：双引号，会引起转义，如"hello \n world"输出为hello 换行 world。

‘’：单引号，可以避免转义，如'hello \n world'输出为hello \n world
```



# 配置文件值注入

**配置文件：**

```yml
person:
  lastName: tom
  age: 18
  birthDay: 2012/09/01
  info: {height: 170, motto: Good morning!}
  friends: [Anna, Bella, Coco]
  dog:
    name: Cindy
    age: 2
```

**JavaBean**

```java
/**
 * Person类
 * 将配置文件中的值映射到当前组件中
 * @ConfigurationProperties： 告诉Spring Boot将本类中的属性和配置文件中相关的配置进行绑定
 * prefix = "person"：将配置文件中person下面的属性进行一一映射
 */
@Component
@Data
@ConfigurationProperties(prefix = "person")
public class Person {
    String lastName;
    Integer age;
    Date birthDay;
    private Map<String, Object> info;
    private List<Object> friends;
    private Dog dog;
}
```

需要导入配置文件处理器，以后编写配置就有提示了

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>
```

#  properties配置文件编码问题

Preferences -> Editor -> File Encodings，将编码方式改为UTF-8。

设置后还乱码的话可以删除application.properties，再新建一个相同的。



#  @ConfigurationProperties和@Value

| 注解                  | `@ConfigurationProperties` | `@Value`     |
| --------------------- | -------------------------- | ------------ |
| 功能                  | 批量注入配置文件中的属性   | 注入单个属性 |
| 松散绑定（松散语法）  | 支持                       | 不支持       |
| SpEL                  | 不支持                     | 支持         |
| JSR303数据校验        | 支持                       | 不支持       |
| 复杂类型封装（如map） | 支持                       | 不支持       |



# @PropertySource和@ImportResource

**@PropertySource**

`@ConfigurationProperties`注解从配置文件中获取值，默认是全局配置文件，配合`@PropertySource`加载指定的配置文件。

```java
@PropertySource(value = {"classpath:person.properties"})
@ConfigurationProperties(prefix = "person")
```

**@ImportResource**

导入Spring的配置文件，让配置文件里面的内容生效。

```java
@ImportResource(locations = {"classpath:beans.xml"})
```

**Spring Boot推荐给容器中添加组件的方式**

Spring Boot推荐使用全注解的方式。

例如：给容器中添加组件，可以使用配置类代替Spring的配置文件，使用`@Bean`给容器中添加组件。

```java
/**
 * @configuration: 指明当前类是一个配置类，用来代替前面的Spring的配置文件
 */
@Configuration
public class MyConfig {

    // 将方法的返回值添加到容器中，容器中这个组件的id就是方法名
    @Bean
    public HelloService helloService() {
        return new HelloService();
    }
}
```



#  配置文件占位符

**RandomValuePropertySource：**配置文件中可以使用随机数

```properties
${random.value}
${random.int}
${random.int}
${random.int(10)}
${random.int[1024,65536]}
```

**属性配置占位符:**

可以在配置文件中引用前面配置过的属性

如果没有配置可以用`${app.name:默认值}`的形式指定默认值

```java
app.name=MyApp
app.description={app.name} is a Spring Boot application
```



# Profile

**多profile文件支持：**

在主配置文件编写的时候，文件名可是是application-{profile}.properties/yml

默认使用application.properties的配置。

**多profile文档块模式**

```properties
spring:
  profiles:
    active: dev
---
server:
  port: 8081

---
server:
  port: 8082
```

**激活方式**

- 在配置文件中指定

```properties
spring.profiles.active=dev
```

- 命令行

```shell
--spring.profiles.active=dev
```

- JVM参数

```shell
-Dspring.profiles.active=dev
```



# 配置文件的加载位置

Spring Boot启动会扫描以下位置的application.properties或者application.yml文件作为Spring Boot的默认配置文件

```
-file: ./config/
-file: ./
-classpath: /config/
-classpath: /
```

优先级从高到低，高优先级配置内容会覆盖低优先级配置内容

可以通过配置`spring.config.location`来改变默认配置文件位置，项目打包好以后，可以使用命令行参数的形式，在启动项目时指定配置文件的新位置，指定的和默认的配置文件会互补配置

```shell
--spring.config.location=xxx/application.properties
```



# 外部配置加载顺序

Spring Boot可以从以下位置加载配置，优先级从高到低，高优先级配置覆盖低优先级配置，所有的配置会形成互补配置。

- 命令行参数

```shell
java -jar myProject.jar --server.port=8080 --server.context-path=/boot
```

- 来自java:comp/evn的JDNI属性

- Java系统属性（`System.getProperties()`)

- 操作系统环境变量

- RandomValuePropertySource配置的`random.*`属性值

- jar包外部的application-{profile}.properties或application.yml（带`spring.profile`）配置文件

- jar包内部的application-{profile}.properties或application.yml（带`spring.profile`）配置文件

- jar包外部的application.properties或application.yml（不带`spring.profile`）配置文件

- jar包内部的application.properties或application.yml（不带`spring.profile`）配置文件

- `@Configuration`注解类上的`@PropertySource`

- 通过`SpringApplication.setDefaultProperties`指定的默认属性



# 自动配置原理

[能配置的属性参照](https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/appendix-application-properties.html#common-application-properties)

1.Spring Boot启动的时加载主配置类，开启了自动配置功能`@EnableAutoConfiguration`

2.`@EnableAutoConfiguration`的作用:

(1)利用`AutoConfigurationImportSelector`给容器中导入一些组件

(2)获取候选配置的关键方法：

```java
List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
```

(3)该方法调用`SpringFactoriesLoader.loadFactoryNames()`

```java
// SpringFactoriesLoader.loadFactoryNames()
扫描所有jar包类路径下META-INF/spring.factories，把扫描到的这些文件的内容包装成properties对象，从properties中取得EnableAutoConfiguration.class类名对应的值，然后把它们添加到容器中
```

(4)即将类路径`META-INF/spring.facories`里面配置的所有`EnableAutoConfiguration`的值加入到了容器中

![image-20200703203903550](https://gitee.com/tongying003/MapDapot/raw/master/img/20200822013640.png)

每一个这样的xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中，用他们来做自动配置；

3.每一个自动配置类进行自动配置功能

4.以HttpEncodingAutoConfiguration为例解释自动配置原理：

```java
// 表示这是一个配置类，也可以给容器中添加组件
@Configuration(proxyBeanMethods = false)
// 启用指定类的ConfigurationProperties功能，从配置文件中，将配置文件中对应的值和ServerProperties绑定起来，并把ServerProperties加入到IOC容器中
@EnableConfigurationProperties(ServerProperties.class)
// Spring底层@Conditional注解，如果满足指定的条件，整个配置类里的配置就会生效；这里是判断当前配置类在Web应用生效
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
// 判断当前项目有没有这个类，CharacterEncodingFilter是Spring MVC中解决乱码的过滤去
@ConditionalOnClass(CharacterEncodingFilter.class)
// 判断配置文件中是否存在某个配置server.servlet.encoding.enabled，matchIfMissing = true，如果不存在，判断也是成立的
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration {

    // 它已经和Spring Boot的配置文件映射了
	private final Encoding properties;

    // 只有一个有参构造器的情况下，参数的值就会从容器中拿
	public HttpEncodingAutoConfiguration(ServerProperties properties) {
		this.properties = properties.getServlet().getEncoding();
	}

    // 给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@Bean
    // 判断容器中不存在这个Bean
	@ConditionalOnMissingBean
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
		return filter;
	}
```

根据当前不同的条件判断，决定这个配置类是否生效。一旦这个配置类生效，这个配置类就会给容器中添加组件，这些组件的属性是从对应的properties类中获取的，这些类中的每一个属性又是和配置文件绑定的。

5.所有能在配置文件中配置的属性都封装在xxxProperties类中，配置文件中能配置的属性就可以参照该功能对应的这个属性类

```java
// 从配置文件中获取指定的值和bean的属性进行绑定
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
```



## Spring Boot精髓

1.Spring Boot启动会加载大量的自动配置类

2.我们先查看需要的功能是否有Spring Boot写好的自动配置类

3.再看自动配置类中配置了哪些组件，如果已经存在我们要用的组件，就不需要再进行配置

4.给容器中自动配置类添加组件的时候会从xxxProperties类中获取某些组件，我们就可以在配置文件中指定这些属性的值。

```
xxxAutoConfiguration: 自动配置类，给容器中添加组件
xxxProperties: 封装配置文件中相关属性
```



## 细节

**`@Conditional`派生注解**

作用：必须是`@Conditional`指定的条件成立，才给容器中添加组件，配置里面的所有内容才生效

| `@Conditional`扩展注解            | 作用（判断是否满足当前指定的条件）               |
| --------------------------------- | ------------------------------------------------ |
| `@ConditionalOnJava`              | 系统的Java版本是否符合要求                       |
| `@ConditionalOnBean`              | 系统中存在指定的Bean                             |
| `@ConditionalOnMissingBean`       | 容器中不存在指定的Bean                           |
| `@ConditionalOnExpression`        | 满足SpEl表达式指定                               |
| `@ConditionalOnClass`             | 系统中有指定的类                                 |
| `@ConditionalOnMissingClass`      | 系统中没有指定的类                               |
| `@ConditionalOnSingleCandidate`   | 系统中只有一个指定的Bean，或者这个Bean是首选Bean |
| `@ConditionalOnProperty`          | 系统中指定的属性是否有指定的值                   |
| `@ConditionalOnSource`            | 类路径下是否存在指定的资源文件                   |
| `@ConditionalOnWebApplication`    | 当前是Web环境                                    |
| `@ConditionalOnNotWebApplication` | 当前不是Web环境                                  |
| `@ConditionalOnJndi`              | JNDI存在指定项                                   |

自动配置类必须在一定的条件下才能生效，如何知道哪些自动配置类生效？

**自动配置报告**

通过启用`debug=true`属性，在控制台打印自动配置报告，这样就可以很方便的知道哪些自动配置类生效。

```properties
# application.properties
debug=true
```

```verilog
============================
CONDITIONS EVALUATION REPORT
============================
    
Positive matches:(自动配置类启用的)
-----------------

   AopAutoConfiguration matched:
      - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)

   AopAutoConfiguration.ClassProxyingConfiguration matched:
      - @ConditionalOnMissingClass did not find unwanted class 'org.aspectj.weaver.Advice' (OnClassCondition)
      - @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)
          
          
          ...
          

Negative matches:（没有启用，没有匹配成功的）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'javax.jms.ConnectionFactory' (OnClassCondition)

   AopAutoConfiguration.AspectJAutoProxyingConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice' (OnClassCondition)   
```



