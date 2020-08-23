---
title: Spring Boot入门
date: 2020-08-17 23:48:32
categories:
- [Spring Boot基础]
tags:
- Spring Boot
- Java
---

# 微服务

每个功能元素最终都是一个可独立替换和独立升级的软件单元。

![20200518121536](https://gitee.com/tongying003/MapDapot/raw/master/img/20200822013324.png)

<!-- more -->

# Hello World探究

认识一下SpringBoot的版本仲裁与场景启动器。

## POM文件

### 父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.12.RELEASE</version>
</parent>
```

它的父项目是：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.1.12.RELEASE</version>
    <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

由该项目来真正管理Spring Boot应用里面的所有依赖版本。可以称为Spring Boot版本仲裁中心，对于在其dependencies里面管理的依赖，我们在导入时不再需要写版本号。

### 导入的依赖

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter-web**

Spring Boot场景启动器，帮助我们导入了web模块正常运行所依赖的组件。

Spring Boot将所有的功能场景都抽取出来，做成一个个的starter（启动器），只需要在项目里引入这些starter，相关场景的所有依赖都会导入进来，可以根据场景选择导入相应的启动器。

## 主程序类

**@SpringBootApplication**

Spring Boot应用标注在某个类上说明这个类是Spring Boot的主配置类，Spring Boot就应该运行这个类的main方法来启动SpringBoot应用。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

**@SpringBootConfiguration**

标注在某个类上，表示这是一个Spring Boot的配置类。其底层是`@Configuration`，用来标注一个配置类，配置类也是容器中的一个组件`@Component`



**@EnableAutoConfiguration**

该注解告诉Spring Boot开启自动配置功能，由Spring Boot帮我们进行自动配置。

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```



**@AutoConfigurationPackage**

将主配置类（@SpringBootApplication标注的类）所在的包及下面所有子包里里面的所有组件扫描到Spring容器。底层是`@Import({Registrar.class})`，Spring的底层注解@Impo给容器中导入一个组件，导入的组件由`{Registrar.class}`获取。



**@Import({AutoConfigurationImportSelector.class})**

导入哪些组件的选择器，将所有需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中。

会给容器中导入非常多的自动配置类（`xxxAutoConfiguration`）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件。 

![20200518191037](https://gitee.com/tongying003/MapDapot/raw/master/img/20200822013205.png)

有了自动配置类，免去了我们手动编写配置注入功能组件等的工作。其底层调用了

```java
SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class, classLoader);
```

Spring Boot在启动时从类路径下的`META-INF/spring.factories` 中获取`EnableAutoConfiguration`指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作。

J2EE的整体解决方案和自动配置都在`spring-boot-autoconfigure-2.1.12.RELEASE.jar`

