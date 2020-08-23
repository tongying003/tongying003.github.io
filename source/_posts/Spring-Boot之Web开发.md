---
title: Spring Boot之Web开发
date: 2020-08-23 01:17:43
categories:
- [Spring Boot基础]
tags:
- Spring Boot
- Java
---



# 静态资源的映射规则

```java
// 可以设置和静态资源有关的参数，缓存时间等
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
```

<!-- more -->

```java
// 配置webjars
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                                             .addResourceLocations("classpath:/META-INF/resources/webjars/")
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                                             .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
    }
}

// 配置欢迎页
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
                                                           FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
        new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
        this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
    welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
    return welcomePageHandlerMapping;
}
```



**所有的`/webjars/`，都去`classpath:/META-INF/resources/webjars/`找资源**

`webjars`：以jar包的方式引入静态资源

[WebJars查询](https://www.webjars.org)

```xml
<!-- 引入jquery-webjar-->
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

![image-20200705110257138](https://gitee.com/tongying003/MapDapot/raw/master/img/20200823224701.png)

在访问的时候只需要写webjars下面资源的名称即可。

例如：访问jquery.js：[localhost:8080/webjars/jquery/3.5.1/jquery.js]()



 **/\*\*访问当前项目的任何资源**

```java
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/", "classpath:/resources/", 	"classpath:/static/", "classpath:/public/" };

/**
 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
 * /resources/, /static/, /public/].
 */
private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```

访问静态资源：[localhost:8080/my.js]()



**欢迎页：静态资源文件夹下所有的index.html页面，被/\*\*映射**

访问欢迎页：[localhost:8080]()



**favicon.ico**

2.2.x前：在静态资源文件下放置favicon.icon即可

2.2.x后：在html中配置，并将favicon放置在指定的路径

```html
 <link rel="icon" type="image/x-icon" href="/favicon.ico" />
```



# 模板引擎

![模版引擎](https://gitee.com/tongying003/MapDapot/raw/master/img/20200823225159.png)

## 引入thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<!-- 切换版本 -->
<properties>
    <java.version>1.8</java.version>
    <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
    <thymeleaf-layout-dialect.version>2.4.1</thymeleaf-layout-dialect.version>
</properties>
```



## thymeleaf语法

```java
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
```

只需要把html页面放在`classpath:/templates/`，thymeleaf就能自动渲染

导入thymeleaf的名称空间

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

### 语法规则

#### th属性

 `th:text`：改变当前元素里面的文本内容

`th:任意html属性`：替换原生属性的值

属性优先级：

| Order |              Feature               | Explanation                         | Attribute                                             |
| :---: | :--------------------------------: | :---------------------------------- | ----------------------------------------------------- |
|   1   |       **Fragment inclusion**       | 片段包含：jsp:include               | `th:insert`<br/>`th:replace`                          |
|   2   |      **Fragment Interation**       | 遍历：c:foreach                     | `th:each`                                             |
|   3   |      **Fragment evaluation**       | 条件判断：c:if                      | `th:if`<br/>`th:unless`<br/>`th:switch`<br/>`th:case` |
|   4   |   **Local variable definition**    | 声明变量：c:set                     | `th:object`<br>`th:with`                              |
|   5   | **General attribute modification** | 任意属性修改<br>支持prepend、append | `th:attr`<br>`th:attrprepend`<br>`th:attrapend`       |
|   6   | **Special attribute modification** | 修改指定属性默认值                  | `th:value`<br>`th:href`<br>`th:src`<br>...            |
|   7   |  **Text(tag body modification)**   | 修改标签体内容                      | `th:text`<br>`th:utext`                               |
|   8   |     **Fragment specification**     | 声明片段                            | `th:fragment`                                         |
|   9   |        **Fragment removal**        | 移除片段                            | `th：remove`                                          |

#### 表达式

- **Simple expressions（表达式语法）**

    - **Variable Expressions: `${…}`**：获取变量值，OGNL

    （1）获取对象的属性、调用方法

    （2）使用内置的基本对象

    ```
    #ctx : the context object. 
    #vars: the context variables. 
    #locale : the context locale. 
    #request : (only in Web Contexts) the HttpServletRequest object. 
    #response : (only in Web Contexts) the HttpServletResponse object. 
    #session : (only in Web Contexts) the HttpSession object. 
    #servletContext : (only in Web Contexts) the ServletContext object.
    ```

    ```html
    <span th:text="${#locale.country}">US</span>.
    ```

    （3）内置的一些工具对象

    ```
    #execInfo : information about the template being processed. #messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax. 
    #uris : methods for escaping parts of URLs/URIs
    #conversions : methods for executing the configured conversion service (if any). 
    #dates : methods for java.util.Date objects: formatting, component extraction, etc. 
    #calendars : analogous to #dates , but for java.util.Calendar objects. 
    #numbers : methods for formatting numeric objects. 
    #strings : methods for String objects: contains, startsWith, prepending/appending, etc. 
    #objects : methods for objects in general. #bools : methods for boolean evaluation. 
    #arrays : methods for arrays. 
    #lists : methods for lists. 
    #sets : methods for sets.
    #maps : methods for maps. 
    #aggregates : methods for creating aggregates on arrays or collections. 
    #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
    ```

    - **Selection Variable Expressions: `*{…}`**：选择表达式，在功能上和`${}`是一样的，补充：配合`th:object`使用。

    如，下面的两种写法功能相同

    ```html
    <div th:object="${session.user}"> 
        <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p> 
        <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p> 
        <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p> 
    </div>
    ```

    ```html
    <div> 
        <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p> 
        <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p> 
        <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p> 
    </div>
    ```

    - **Message Expressions:** **`#{…}`** ：获取国际化内容
    - **Link URL Expressions:** **`@{…}`** ：定义URL

    ```html
    <!-- Will produce 'http://localhost:8080/gtvg/order/details?orderId=3' (plus rewriting) --> 
    <a href="details.html" th:href="@{http://localhost:8080/gtvg/order/details(orderId=${o.id})}">view</a> 
    <!-- Will produce '/gtvg/order/details?orderId=3' (plus rewriting) --> 
    <a href="details.html" th:href="@{/order/details(orderId=${o.id})}">view</a> 
    <!-- Will produce '/gtvg/order/3/details' (plus rewriting) --> 
    <a href="details.html" th:href="@{/order/{orderId}/details(orderId=${o.id})}">view</a>
    <!--多个参数-->
    <a href="detail.html" th:href="@{/order/process(execId=${execId},execType='FAST')}">view</a>
    ```

    - **Fragment Expressions:** **`~{…}`** ：片段引用表达式

- **Literals（字面量）** 

    - **Text literals:** **`'one text’`** **,**` **'Another one!’`** **,…** 

    - **Number literals:** **`0`** **,** **`34`** **,** **`3.0`** **,** **`12.3`** **,…** 

    - **Boolean literals:** **`true`** **,** **`false`** 

    - **Null literal:** **`null`** 

    - **Literal tokens:** **`one`** **,** **`sometext`** **,** **`main`** **,…** 

- **Text operations:（文本操作）** 

    - **String concatenation:** **`+`** 

    - **Literal substitutions:** **`|The name is ${name}|`** 

- **Arithmetic operations:（数学运算）** 

    - **Binary operators:** **`+`** **,** **`-`** **,** **`*`** **,** **`/`** **,** **`%`** 

    - **Minus sign (unary operator):** **`-`** 

- **Boolean operations:（布尔运算）** 

    - **Binary operators:** **`and`** **,** **`or`** 
    - **Boolean negation (unary operator):** **`!`** **,** **`not`** 

- **Comparisons and equality:（比较运算）** 

    - **Comparators:** **`>`** **,** **`<`** **,** **`>=`** **,** **`<=`** **(** **`gt`** **,** **`lt`** **,** **`ge`** **,** **`le`** **)** 

    - **Equality operators:** **`==`** **,** **`!=`** **(** **`eq`** **,** **`ne`** **)** 

- **Conditional operators:（条件运算）** 

    - **If-then:** **`(if) ? (then)`** 

    - **If-then-else:** **`(if) ? (then) : (else)`** 

    - **Default:** **`(value) ?: (defaultvalue)`** 

- **Special tokens:（特殊操作）** 

    - **No-Operation:** **`_`**

