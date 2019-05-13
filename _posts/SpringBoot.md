title: SpringBoot入门
---
## springBoot
### 1、简介
1）简化spring应用开发的一个框架
2）整个Spring技术栈的一个大整合
3）J2ee开发的一站式框架

### 2、优点：
1）快速创建独立运行的spring项目以及与主流框架集成
2）使用嵌入式的Servlet容器，应用无需打包成war包
3）starters自动依赖与版本控制
4）大量的自动配置、简化开发，也可修改默认值
5）无需配置xml，无代码生成，开箱即用
6）准生产环境的运行时应用监控
7）与云计算的天然集成

### 3、微服务
1）微服务：架构风格（服务微化）
2）一个应用应该是一组小型服务；可以通过HTTP方式进行互通
3）每一个功能元素最终都是一个可独立替换和独立升级的软件单元

详细参照微服务文档：https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa

### 4、一个简单的例子 HelloWorld
一个功能：
浏览器发送hello请求，服务器接受请求并处理，响应Hello World字符串；
1）、创建一个maven工程；（jar）

2）、导入spring boot相关的依赖
```java
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies
```
3)、编写一个主程序：启动Spring Boot应用
```java
/**
 * @Author: cuzz
 * @Date: 2018/9/20 18:06
 * @Description: @SpringBootApplication 来标注一个主程序，说明这是一个SpringBoot应用
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        
        // Spring应用启动起来
        SpringApplication.run(Application.class, args);
    }
}
```

4）、编写相关的Controller、Service
```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

5）、运行主程序测试
（1）到主程序里运行main()方法，直接右键runApplication
（2）在浏览器输入主机端口号后输一个"hello"

6、简化部署
在pom/xml文件引用插件
```java
 <!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```
导入这个maven插件，利用idea打包，生成的jar包，可以使用java -jar xxx.jar启动
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootenter1.png?raw=true)
注：Spring Boot 使用嵌入式的Tomcat无需再配置Tomcat

## 原理
1、pom.xml
1）其中有一个父项目，这个父项目又依赖于spring-boot-starter-parent
```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
```
2）spring-boot-starter-parent中又有一个父项目
```java
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>1.5.9.RELEASE</version>
  <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```
它依赖于spring-boot-dependencies

3）spring-boot-dependencies中有基本全部的依赖组件
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/springbootenter2.png?raw=true)
由它来真正管理springboot应用里面的所有依赖版本，是springboot的版本仲裁中心，以后导入依赖默认不需要写明版本号，若是没有的依赖需要自己写明版本号。

2、启动器
pom.xml导入的依赖spring-boot-starter-web
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
1）spring-boot-starter：springboot场景启动器，帮我们导入了web模块正常运行所依赖的组件
```java
<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
		</dependency>
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
		</dependency>
	</dependencies>
```
Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来，要用什么功能就导入什么场景的启动器

3、主程序类，主入口类
```java
/**
 * @Author: cuzz
 * @Date: 2018/9/20 18:06
 * @Description: @SpringBootApplication 来标注一个主程序，说明这是一个SpringBoot应用
 */
@SpringBootApplication
public class Application {

    public static void main(String[] args) {

        // Spring应用启动起来
        SpringApplication.run(Application.class, args);
    }
}
```

@SpringBootApplication : Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

1）@SpringBootConfiguration : Spring Boot的配置类，标注在某个类上，表示这是一个Spring Boot的配置类

里面有注解@Configuration : 配置类上来标注这个注解，配置类也是容器中的一个组件@Component

2）@EnableAutoConfiguration：开启自动配置功能

以前我们需要配置的东西，Spring Boot帮我们自动配置；@EnableAutoConfiguration告诉SpringBoot开启自动配置功能；这样自动配置才能生效；

里面有注解
```java
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
```

@AutoConfigurationPackage：自动配置包

@Import(AutoConfigurationPackages.Registrar.class)：
Spring的底层注解@Import，给容器中导入一个组件；导入的组件由AutoConfigurationPackages.Registrar.classz这个类来决定导入哪个注解；
将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；

@Import(EnableAutoConfigurationImportSelector.class)，给容器中导入组件

EnableAutoConfigurationImportSelector：导入哪些组件的选择器；
将所有需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中； 会给容器中导入非常多的自动配置类（xxxAutoConfiguration），就是给容器中导入这个场景需要的所有组件，并配置好这些组件； 自动配置类
有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

调用了SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader)；

### Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；

以前我们需要自己配置的东西，自动配置类都帮我们；

J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-1.5.9.RELEASE.jar；

## 使用Spring Initializer快速创建Spring Boot项目

1、IDEA：使用 Spring Initializer快速创建项目

1）IDE都支持使用Spring的项目创建向导快速创建一个Spring Boot项目；

2）选择我们需要的模块；向导会联网创建Spring Boot项目；

3）默认生成的Spring Boot项目；

4）主程序已经生成好了，我们只需要我们自己的逻辑
（1）resources文件夹中目录结构
（2）static：保存所有的静态资源； js css images；
（3）templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）；
（4）application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；
2、STS使用 Spring Starter Project快速创建项目