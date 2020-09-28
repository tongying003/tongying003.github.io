---
title: Spring Boot与数据访问
date: 2020-09-03 23:50:24
categories:
- Spring Boot基础
tags:
- Spring Boot
- Java
---

# JDBC

导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

数据源配置

```yml
spring:
  datasource:
    username: root
    password: xxxxxxxx
    url: jdbc:mysql://127.0.0.1/jdbc
    driver-class-name: com.mysql.cj.jdbc.Driver
```

效果：

​		默认使用com.zaxxer.hikari.HikariDataSource作为数据源

​		数据源相关配置在**DataSourceProperties**中

<!-- more -->

**自动配置原理：**

1. 参考**DataSourceConfiguration**，根据配置创建数据源，默认使用**Hikari**连接池，可以使用`spring.datasource.type`指定自定义的数据源类型

2. Spring Boot默认支持的数据源有

    ```java
    org.apache.tomcat.jdbc.pool.DataSource
    com.zaxxer.hikari.HikariDataSource
    org.apache.commons.dbcp2.BasicDataSource
    ```

3. 自定义数据源类型

    ```java
    /**
    	 * Generic DataSource configuration.
    	 */
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnMissingBean(DataSource.class)
    @ConditionalOnProperty(name = "spring.datasource.type")
    static class Generic {
    
        @Bean
        DataSource dataSource(DataSourceProperties properties) {
            // 使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
            return properties.initializeDataSourceBuilder().build();
        }
    
    }
    ```

4. DataSourceInitializer

    **方法：`runScripts`**

    作用：运行sql语句

    <br>

    **方法：`createSchema`**

    作用：调用runScripts运行建表语句

    默认文件名：`classpath*:schema-all.sql`

    也可用spring.datasource.schema指定路径和文件名

    <br>

    **方法：`initSchema`**

    默认作用：调用runScripts运行插入数据的sql语句

    文件名：`classpath*:data.sql`

    也可以用spring.datasource.data指定路径和文件名

    <br>

    需要打开如下配置才能自动调用

    ```properties
    initialization-mode: always
    ```

5. 操作数据库：自动配置了JdbcTemplate操作数据库。

```java
@Controller
public class HelloController {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @ResponseBody
    @RequestMapping("/query")
    public Map<String, Object> query() {
        List<Map<String, Object>> list = jdbcTemplate.queryForList("select * from department");
        return list.get(0);
    }

}
```



#  整合Druid数据源

 ```java
@Configuration
public class DruidConfig {


    //数据源属性配置
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid() {
        return new DruidDataSource();
    }

    // 配置druid的监控
    // 1. 配置一个管理后台的servlet
    @Bean
    public ServletRegistrationBean<StatViewServlet> statViewServlet() {
        ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");
        Map<String, String> initParams = new HashMap<>();
        initParams.put("loginUsername", "admin");
        initParams.put("loginPassword", "123456");
        initParams.put("allow", "");   // 默认允许所有访问
        initParams.put("deny", "192.168.102.11");

        bean.setInitParameters(initParams);
        return bean;
    }

    // 2. 配置一个web的监控filter
    @Bean
    FilterRegistrationBean<WebStatFilter> statViewFilter() {
        FilterRegistrationBean<WebStatFilter> bean = new FilterRegistrationBean<>();
        bean.setFilter(new WebStatFilter());
        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*.js, *.css, /druid/*");
        bean.setUrlPatterns(Arrays.asList("/*"));
        bean.setInitParameters(initParams);
        return bean;
    }
}
 ```



# 整合Mybatis

导入依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.3</version>
</dependency>
```

![image-20200829160911280](https://gitee.com/tongying003/MapDapot/raw/master/img/20200903235426.svg)

**步骤：**

配置数据源（见上节）

给数据库建表

创建Java Bean

## 注解版

```java
// 指定这是一个操作数据库的mapper
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id = #{id}")
    Department getDeptById(Integer id);

    @Delete("delete from department where id = #{id}")
    int deleteDeptById(Integer id);

    @Options(useGeneratedKeys = true, keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    int insertDept(Department department);

    @Update("update department set departmentName = #{departmentName}")
    int updateDepat(Department department);
}
```

自定义MyBatis的配置规则：

给容器中添加一个ConfigurationCustomizer

```java
@Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return configuration -> configuration.setMapUnderscoreToCamelCase(true);
    }
}
```

使用@MapperScan注解扫描所有的Mapper接口

```java
@MapperScan(basePackages = "com.javalearning.springboot.mapper")
@SpringBootApplication
public class SpringBoot06DataMybatisApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
    }
}
```



##  配置文件版

```yaml
mybatis:
  # 指定全局配置文件的位置
  config-location: classpath:mybatis/mybatis-config.xml
  # 指定sql映射文件的位置
  mapper-locations: classpath:mybatis/mapper/*.xml
```



# 整合JPA

## Spring Boot JPA简介

Spring Data

Spring Data项目是目前为了简化构建基于Spring框架应用的数据库访问技术，包括非关系数据库、Map-Reduce框架、云数据服务等等，另外也包含对关系型数据库的访问支持。

Spring Data包含多个子项目：

- Spring Data Commons
- Spring Data JPA
- Spring Data KeyValue
- Spring Data MongoDB
- Spring Data LDAP
- Spring Data Gemfire
- Spring Data REST
- Spring Data Redis
- Spring Data for Apache Cassandra
- Spring Data for Apache Solr
- Spring Data Couchbase(community module)
- Spring Data Elasticsearch(community module)
- Spring Data Neo4j(community module)

**Spring Data特点**

Spring Data为我们提供使用同一的API来对数据访问层进行操作，这主要是Spring Data Commons项目来实现的。Spring Data Commons让我们在使用关系型或者非关系型数据访问技术时都基于Spring提供的统一标准，标准包含了CRUD、排序和分页等相关操作。

**统一的Repository接口**

```java
// 统一接口
Repository<T, ID extends Serializable>
    
// 基于乐观锁机制
RevisionRepository<T, ID extends Serializable, N extends Number & Comparable<N>>

// 基于CRUD操作    
CrudRepository<T, ID extends Serializable>
    
// 基本CRUD操作
PagingAndSortingRepository<T, ID extends Serializable>
```



**提供数据访问模板类**

如MongoTemplate、RedisTemplate

**JPA与Spring Data**

1）编写接口继承JpaRepository即有crud及分页等基本功能

2）在接口中只需要声明符合规范的方法，即拥有对应的功能

3）@Query自定义查询，定制查询SQL

4）Specifications查询（Spring Data JPA支持JPA2.0的Criteria查询）

![SpringDataJPA](https://gitee.com/tongying003/MapDapot/raw/master/img/20200903235426.svg)

## 整合JPA

编写一个实体类和数据表进行映射，并且配置好映射关系

```java
@Entity         // 告诉JPA这是一个实体类
@Table(name = "tbl_user")       // 用来指定与哪个数据表对应，如果省略默认表明就是user
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer Id;
    @Column(name = "last_name", length = 50)
    private String lastName;
    @Column     // 省略列名就是属性名
}
```

编写一个Dao层接口来操作实体类对应的数据表（Repository）

```java
// 继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User, Integer> {
}
```

基本的配置JpaProperties

```yaml
spring:
  jpa:
    hibernate:
    # 更新或创建表结构
      ddl-auto: update
    # 控制台显示SQL
    show-sql: true
    database: mysql
```

