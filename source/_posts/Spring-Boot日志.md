---
title: Spring Boot日志
date: 2020-08-22 18:01:54
categories:
- [Spring Boot基础]
tags:
- Spring Boot
- Java
---

# 日志框架的分类与选择

**常见的日志门面**

- JCL（Jakarta Commons Logging）

- SLF4j（Simple Logging Facade for Java）

- jboss-logging

**常见的日志实现**

- Log4j

- JUL（java.util.logging）

- log4j2

- Logback

**日志框架选择**

Spring Boot在框架内容部使用JCL，spring-boot-starter-logging采用了==slf4j + logback==的形式，Spring Boot也能自动适配（JUL、Log4j2、logback）并简化配置。

<!-- more -->

# SLF4j使用原理

**如何在系统中使用SLF4j**

在开发的时候，日志记录方法的调用，不应该直接调用日志的实现类，而是调用日志抽象层里面的方法。

给系统导入slf4j和logback的jar包

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

图示：

![click to enlarge](https://gitee.com/tongying003/MapDapot/raw/master/img/20200822181558.png)



**遗留问题**

统一日志记录：即使项目中使用到了其他框架也统一使用slf4j进行输出

![click to enlarge](https://gitee.com/tongying003/MapDapot/raw/master/img/20200822181800.png)

如何让系统中所有的日志都统一到slf4j？

1）将系统中其他日志框架先排除出去

2）用中间包来替换原有的日志框架

3）导入slf4j其他的实现



# Spring Boot日志关系

Spring Boot日志功能：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
  <version>2.3.1.RELEASE</version>
  <scope>compile</scope>
</dependency>
```

底层依赖关系：

![image-20200704131902902](https://gitee.com/tongying003/MapDapot/raw/master/img/20200822181924.png)

1）Spring Boot底层也是使用slf4j + logback的方式进行日志记录

2）Spring Boot也把其他的日志都替换成了slf4j

3）中间替换包`jul-to-slf4j`、`log4j-to-slf4j`

4）如果在项目中引入了其他框架，一定要把这个框架的默认日志依赖移除掉，不然中间替换包与默认日志依赖不就冲突了吗？

例如：

Spring5并没有使用`commons-logging`，而是仿照`commons-logging`实现的`spring-jcl`，只不过包名是`org.apache.commons.logging`而已，使其可以支持`slf4j`，所以Spring5并不需要排除掉`commons.logging`

![image-20200704132720699](https://gitee.com/tongying003/MapDapot/raw/master/img/20200823010317.png)

# 日志使用

## 默认配置

Spring Boot默认帮我们配置好了日志

```java
@SpringBootTest
class SpringBoot03LoggingApplicationTests {
    Logger logger = LoggerFactory.getLogger(getClass());
    @Test
    void contextLoads() {
        logger.trace("这是trace日志");
        logger.debug("这是debug日志");
        logger.info("这是info日志");
        logger.warn("这是warn日志");
        logger.error("这是error日志");
    }
}
```

- 日志级别：`trace < debug < info < warn < error`
- Spring默认规定的日志级别（root）：`info`

- 日志的路径

```properties
# 在指定的路径下生成日志文件，若不指定路径，则在项目路径下生成
logging.file.name=mylog.log

# 在指定路径下生成名为springboot.log日志文件
logging.file.path=/springboot/log

# 若都不指定，只在控制台输出
# 若都指定，logging.file.name生效
```

| `logging.file.name` | `logging.file.path` | Example    | Description                                                  |
| :------------------ | :------------------ | :--------- | :----------------------------------------------------------- |
| *(none)*            | *(none)*            |            | Console only logging.                                        |
| Specific file       | *(none)*            | `my.log`   | Writes to the specified log file. Names can be an exact location or relative to the current directory. |
| *(none)*            | Specific directory  | `/var/log` | Writes `spring.log` to the specified directory. Names can be an exact location or relative to the current directory. |

- 日志模版

```properties
# 控制台日志输出的格式
logging.pattern.console=%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n

# 日志文件中输出个格式
logging.pattern.file=%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
```

```properties
%d：时间
%thread：线程名
%-5level：级别从左显示5个字符宽度
%logger{36}：表示logger名字最长50个字符，否则按照句点分割
%msg：日志信息
%n：换行符
```

## 指定配置

[官网文档](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logging)

给类路径下放上每个日志框架自己的配置文件即可，Spring Boot就不使用默认配置了。

| Logging System          | Customization                                                |
| :---------------------- | :----------------------------------------------------------- |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

`logback.xml`：配置文件直接被日志框架识别

`logback-spring.xml`：日志框架不直接加载日志的配置项，由Spring Boot解析日志的配置，推荐使用。可以使用Spring Boot的高级特性，如`<springProfile>`标签。

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

logback.xml配置示例：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒当scan为true时，此属性生效。默认的时间间隔为1分钟。
debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-->
<configuration scan="false" scanPeriod="60 seconds" debug="false">
    <!-- 定义日志的根目录 -->
    <property name="LOG_HOME" value="/Users/tongying/Workspace/springboot" />
    <!-- 定义日志文件名称 -->
    <property name="appName" value="javalearning-springboot"></property>
    <!-- ch.qos.logback.core.ConsoleAppender 表示控制台输出 -->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <!--使用logback-spring.xml时，支持Spring Boot高级特性springProfile，在不同的Profile生效-->
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} --> [%thread] %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} == [%thread] %-5level %logger{50} - %msg%n</pattern>
            </springProfile>

        </layout>
    </appender>

    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
    <appender name="appLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 指定日志文件的名称 -->
        <file>${LOG_HOME}/${appName}.log</file>
        <!--
        当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名
        TimeBasedRollingPolicy： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--
            滚动时产生的文件的存放位置及文件名称 %d{yyyy-MM-dd}：按天进行日志滚动
            %i：当文件大小超过maxFileSize时，按照i进行文件滚动
            -->
            <fileNamePattern>${LOG_HOME}/${appName}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <!--
            可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每天滚动，
            且maxHistory是365，则只保存最近365天的文件，删除之前的旧文件。注意，删除旧文件是，
            那些为了归档而创建的目录也会被删除。
            -->
            <MaxHistory>365</MaxHistory>
            <!--
            当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动 注意此处配置SizeBasedTriggeringPolicy是无法实现按文件大小进行滚动的，必须配置timeBasedFileNamingAndTriggeringPolicy
            -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 日志输出格式： -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n</pattern>
        </layout>
    </appender>

    <!--
		logger主要用于存放日志对象，也可以定义日志类型、级别
		name：表示匹配的logger类型前缀，也就是包的前半部分
		level：要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR
		additivity：作用在于children-logger是否使用 rootLogger配置的appender进行输出，
		false：表示只用当前logger的appender-ref，true：
		表示当前logger的appender-ref和rootLogger的appender-ref都有效
    -->
    <!-- hibernate logger -->
    <logger name="com.atguigu" level="debug" />
    <!-- Spring framework logger -->
    <logger name="org.springframework" level="debug" additivity="false"></logger>



    <!--
    root与logger是父子关系，没有特别定义则默认为root，任何一个类只会和一个logger对应，
    要么是定义的logger，要么是root，判断的关键在于找到这个logger，然后判断这个logger的appender和level。
    -->
    <root level="info">
        <appender-ref ref="stdout" />
        <appender-ref ref="appLogAppender" />
    </root>
</configuration>
```



## 切换日志框架

可以按照slf4j的日志适配图，进行相关的切换

**slf4j + log4j**

这样做并没有什么意义

排除掉`logback`、`log4j-to-slf4j`，引入`slf4j-log4j12`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>logback-classic</artifactId>
            <groupId>ch.qos.logback</groupId>
        </exclusion>
        <exclusion>
            <artifactId>log4j-to-slf4j</artifactId>
            <groupId>org.apache.logging.log4j</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
</dependency>
```

**切换为log4j2**

排除掉`spring-boot-starter-logging`，引入`spring-boot-starter-log4j2`即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

