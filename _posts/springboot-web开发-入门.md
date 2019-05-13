title: SpringBoot与Web开发入门
---
### 1、简介

使用SpringBoot；

1）、创建SpringBoot应用，选中我们需要的场景模块；

2）、SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来

3）、自己编写业务代码；

自动配置原理：
这个场景SpringBoot帮我们配置了什么？能不能修改？能修改哪些配置？能不能扩展？xxx

xxxxAutoConfiguration：帮我们给容器中自动配置组件；
xxxxProperties:配置类来封装配置文件的内容；

2、SpringBoot对静态资源的映射规则；
```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware {
  //可以设置和静态资源有关的参数，缓存时间等
 ```
 
这个类WebMvcAuotConfiguration：
```java
org\springframework\boot\autoconfigure\web\servlet\WebMvcAutoConfiguration.java

		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Integer cachePeriod = this.resourceProperties.getCachePeriod();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler("/webjars/**")
								.addResourceLocations(
										"classpath:/META-INF/resources/webjars/")
						.setCachePeriod(cachePeriod));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
          	// 静态资源文件夹映射
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(
						registry.addResourceHandler(staticPathPattern)
								.addResourceLocations(
										this.resourceProperties.getStaticLocations())
						.setCachePeriod(cachePeriod));
			}
		}

         // 配置欢迎页映射
		@Bean
		public WelcomePageHandlerMapping welcomePageHandlerMapping(
				ResourceProperties resourceProperties) {
			return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage(),
					this.mvcProperties.getStaticPathPattern());
		}

         // 配置喜欢的图标
		@Configuration
		@ConditionalOnProperty(value = "spring.mvc.favicon.enabled", matchIfMissing = true)
		public static class FaviconConfiguration {

			private final ResourceProperties resourceProperties;

			public FaviconConfiguration(ResourceProperties resourceProperties) {
				this.resourceProperties = resourceProperties;
			}

			@Bean
			public SimpleUrlHandlerMapping faviconHandlerMapping() {
				SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
				mapping.setOrder(Ordered.HIGHEST_PRECEDENCE + 1);
              	 // 所有  **/favicon.ico 
				mapping.setUrlMap(Collections.singletonMap("**/favicon.ico",
						faviconRequestHandler()));
				return mapping;
			}

			@Bean
			public ResourceHttpRequestHandler faviconRequestHandler() {
				ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
				requestHandler
						.setLocations(this.resourceProperties.getFaviconLocations());
				return requestHandler;
			}

		}
```
1）、所有 /webjars/** ，都去 classpath:/META-INF/resources/webjars/ 找资源；

​	webjars：以jar包的方式引入静态资源；

参考： http://www.webjars.org/

![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootweb1.png?raw=true)
访问地址：
localhost:8080/webjars/jquery/3.3.1/jquery.js

<!--引入jquery-webjar-->在访问的时候只需要写webjars下面资源的名称即可
```java
		<dependency>
			<groupId>org.webjars</groupId>
			<artifactId>jquery</artifactId>
			<version>3.3.1</version>
		</dependency>
```
2）、"/**" 访问当前项目的任何资源，都去（静态资源的文件夹）找映射
```java
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
"/"：当前项目的根路径
```
localhost:8080/abc === 去静态资源文件夹里面找abc

3）、欢迎页； 静态资源文件夹下的所有index.html页面；被"/**"映射；

​	localhost:8080/ 找index页面

4）、所有的 **/favicon.ico 都是在静态资源文件下找；
注：在配置文件中改变静态资源的访问路径后，原来默认的静态资源访问路径就不可以访问了。

3、模板引擎
新建项目如果是jar包，并且使用嵌入式的tomacat，则不支持jsp。

模板引擎：JSP、Velocity、Freemarker、Thymeleaf
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootweb2.png?raw=true)

SpringBoot推荐的Thymeleaf；
语法更简单，功能更强大；

1、引入thymeleaf；

thymeleaf官网：https://www.thymeleaf.org/
``` java
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
          	//2.1.6版本太低
</dependency>
```

切换thymeleaf版本
```java
 <properties>
        <java.version>1.8</java.version>
        <!-- 布局功能的支持程序  thymeleaf3主程序  layout2以上版本 -->
        <!-- thymeleaf2   layout1-->
        <thymeleaf-spring5.version>3.0.9.RELEASE</thymeleaf-spring5.version>
        <thymeleaf-layout-dialect.version>2.2.2</thymeleaf-layout-dialect.version>
    </properties>
```
  
2、Thymeleaf使用
```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

	private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");

	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
  	//
只要我们把HTML页面放在classpath:/templates/，thymeleaf就能自动渲染；
```

使用：

1、导入thymeleaf的名称空间
```java
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

2、使用thymeleaf语法；
```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
</body>
</html>
```

3、语法规则

1）、th:text；改变当前元素里面的文本内容；

​	th：任意html属性；来替换原生属性的值
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootweb3.png?raw=true)

2）、表达式
```java
Simple expressions:（表达式语法）
a. Variable Expressions: ${...}：获取变量值；OGNL；
	1）、获取对象的属性、调用方法
	2）、使用内置的基本对象：
    #ctx : the context object.
    #vars: the context variables.
    #locale : the context locale.
    #request : (only in Web Contexts) the HttpServletRequest object.
    #response : (only in Web Contexts) the HttpServletResponse object.
    #session : (only in Web Contexts) the HttpSession object.
    #servletContext : (only in Web Contexts) the ServletContext object.
	${session.foo}
    3）、内置的一些工具对象：
        #execInfo : information about the template being processed.
        #messages : methods for obtaining externalized messages inside variables expressions,
        #          in the same way as they would be obtained using #{…} syntax.
        #uris : methods for escaping parts of URLs/URIs
        #conversions : methods for executing the configured conversion service (if any).
        #dates : methods for java.util.Date objects: formatting, component extraction, etc.
        #calendars : analogous to #dates , but for java.util.Calendar objects.
        #numbers : methods for formatting numeric objects.
        #strings : methods for String objects: contains, startsWith, prepending/appending, etc.
        #objects : methods for objects in general.
        #bools : methods for boolean evaluation.
        #arrays : methods for arrays.
        #lists : methods for lists.
        #sets : methods for sets.
        #maps : methods for maps.
        #aggregates : methods for creating aggregates on arrays or collections.
        #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

b. Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充：配合 th:object="${session.user}：
        <div th:object="${session.user}">
        <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
        <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
        <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
        </div>
    
c. Message Expressions: #{...}：获取国际化内容
d. Link URL Expressions: @{...}：定义URL；
    		@{/order/process(execId=${execId},execType='FAST')}
f. Fragment Expressions: ~{...}：片段引用表达式
    		<div th:insert="~{commons :: main}">...</div>
    		
Literals（字面量）
    Text literals: 'one text' , 'Another one!' ,…
    Number literals: 0 , 34 , 3.0 , 12.3 ,…
    Boolean literals: true , false
    Null literal: null
    Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:
    No-Operation: _ 
```

## 4、SpringMVC自动配置

https://docs.spring.io/spring-boot/docs/1.5.10.RELEASE/reference/htmlsingle/#boot-features-developing-web-applications

1. Spring MVC auto-configuration

Spring Boot 自动配置好了SpringMVC

以下是SpringBoot对SpringMVC的默认配置:（WebMvcAutoConfiguration）
```java
Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.
自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发或重定向页面））
ContentNegotiatingViewResolver：组合所有的视图解析器的；
如何定制：我们可以自己给容器中添加一个视图解析器；自动的将其组合进来；

Support for serving static resources, including support for WebJars (see below).静态资源文件夹路径,webjars

Static index.html support. 静态首页访问

Custom Favicon support (see below). favicon.ico

自动注册了 of Converter, GenericConverter, Formatter beans.

Converter：转换器； public String hello(User user)：类型转换使用Converter
Formatter 格式化器； 2017.12.17===Date；
```
配置格式化器
```java
@Bean
@ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")//在文件中配置日期格式化的规则
public Formatter<Date> dateFormatter() {
    return new DateFormatter(this.mvcProperties.getDateFormat());//日期格式化组件
}
```

注：自己添加的格式化器转换器，我们只需要放在容器中即可
```java
Support for HttpMessageConverters (see below).//消息转换器
    HttpMessageConverter：SpringMVC用来转换Http请求和响应的；User---Json；
HttpMessageConverters             是从容器中确定；获取所有的HttpMessageConverter；
自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component）


Automatic registration of MessageCodesResolver (see below).定义错误代码生成规则

Automatic use of a ConfigurableWebBindingInitializer bean (see below).
我们可以配置一个ConfigurableWebBindingInitializer来替换默认的；（添加到容器）
作用：
初始化WebDataBinder；
请求数据=====JavaBean；
```
在springboot的org.springframework.boot.autoconfigure.web包中：有web的所有自动配置场景；
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootweb4.png?raw=true)

2、扩展SpringMVC
1、方法：
If you want to keep Spring Boot MVC features, and you just want to add additional MVC configuration (interceptors, formatters, view controllers etc.) you can add your own @Configuration class of type WebMvcConfigurerAdapter, but without @EnableWebMvc. If you wish to provide custom instances of RequestMappingHandlerMapping, RequestMappingHandlerAdapter or ExceptionHandlerExceptionResolver you can declare a WebMvcRegistrationsAdapter instance providing such components.

If you want to take complete control of Spring MVC, you can add your own @Configuration annotated with @EnableWebMvc.

2、扩展
```java
    <mvc:view-controller path="/hello" view-name="success"/>
    <mvc:interceptors>//拦截器
        <mvc:interceptor>
            <mvc:mapping path="/hello"/>
            <bean></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

编写一个配置类（@Configuration），是WebMvcConfigurerAdapter类型；不能标注@EnableWebMvc;

既保留了所有的自动配置，也能用我们扩展的配置；
```java
/**
 * @Author: cuzz
 * @Date: 2018/9/21 14:29
 * @Description:
 */
@Configuration
// WebMvcConfigurerAdapter过时,使用WebMvcConfigurer接口
public class MyMvcConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // 浏览器发送 /cuzz 请求来到 success
        registry.addViewController("/cuzz").setViewName("success");
    }
}
```

原理：

​	1）、WebMvcAutoConfiguration是SpringMVC的自动配置类

​	2）、在做其他自动配置时会导入；@Import(EnableWebMvcConfiguration.class)
```java
    @Configuration
	public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
      private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

	 //从容器中获取所有的WebMvcConfigurer
      @Autowired(required = false)
      public void setConfigurers(List<WebMvcConfigurer> configurers) {
          if (!CollectionUtils.isEmpty(configurers)) {
              this.configurers.addWebMvcConfigurers(configurers);
            	//一个参考实现；将所有的WebMvcConfigurer相关配置都来一起调用；  
            	@Override
             	// public void addViewControllers(ViewControllerRegistry registry) {
              	// 	    for (WebMvcConfigurer delegate : this.delegates) {
               	//      delegate.addViewControllers(registry);
               	//   }
              }
          }
	}
```

​	3）、容器中所有的WebMvcConfigurer都会一起起作用；

​	4）、我们的配置类也会被调用；

​	效果：SpringMVC的自动配置和我们的扩展配置都会起作用；

3、全面接管SpringMVC；

SpringBoot对SpringMVC的自动配置不需要了，所有都是我们自己配置；所有的SpringMVC的自动配置都失效了

我们需要在配置类中添加@EnableWebMvc即可；
```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@EnableWebMvc
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
       // super.addViewControllers(registry);
        //浏览器发送 /atguigu 请求来到 success
        registry.addViewController("/atguigu").setViewName("success");
    }
}
```
原理：

为什么@EnableWebMvc自动配置就失效了；

1）@EnableWebMvc的核心
```java
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
```
2）、
```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```
3）、
```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
// 容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```
4）、@EnableWebMvc将WebMvcConfigurationSupport组件导入进来；配置类自动失效

5）、导入的WebMvcConfigurationSupport只是SpringMVC最基本的功能；

### 5、如何修改SpringBoot的默认配置

模式：

​	1）、SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

​	2）、在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置

​	3）、在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置