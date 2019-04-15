title: Spring的bean管理（注解方式）
---
## 注解
1、代码里面的特殊标记，使用注解可以完成某些功能
2、注解写法：@注解名称（属性名称=属性值）
3、注解使用在类、方法、属性上面

### 注解开发准备
1、创建类User
```java 
package com

public class User{
    public void add(){
        System.out.println("add.......");
    }
}
```
2、创建配置文件，引入约束
（1）IOC基本功能，引入beans约束
（2）IOC注解开发，还需引入spring-context约束

3、
```java
<!---开启注解扫描
    （1）到包里面扫描类、方法、属性上面是否有注解
--->
<context:component-scan base-package="com"></context:component-scan>
<!---base-package属性为要开启注解扫描的包--->

<!---
    只扫描属性上面是否有注解
--->
<context:annotation-config></context:annotation-config>
```

## 注解创建对象
```java 
package com

//在类上方使用注解创建对象，相当于<bean id="user" class="com.User"></bean>
@Component(value="user")
public class User{
    public void add(){
        System.out.println("add.......");
    }
}
```
测试类：
```java
public class TestIOC{
    @Test
    public void testUser(){
        //1、加载spring配置文件，扫描注解
        ApplicationContext context=new ClassPathXmlApplicationContext("bean.xml");

        //2、得到注解创建的对象
        User user=(User)context.getBean("user");
        user.add();
    }
}
```

### 创建对象的四个注解
1、@Component
2、@Controller：web层
3、@Service：业务层
4、@Repository：持久层
目前四个注解功能一样，都是创建对象，后三个注解为了让类标注本身的用途清晰，spring后续版本会对其增强。

### 注解创建多实例对象
```java 
package com

//@Scope(value="propertype)指明创建多实例对象
@Component(value="user")
@Scope(value="propertype")
public class User{
    public void add(){
        System.out.println("add.......");
    }
}
```

## 注解注入属性
1、创建UserService类，用注解@Component(value="userService")创建该类对象userService
```java 
@Component(value="userService")
public class UserService{

    public void add(){
        System.out.println("service........");
        //在service中得到Dao对象，才能调用Dao的方法
        userDao.add();
    }
}
```
2、创建UserDao类，用注解@Component(value="userDao")创建该类对象userDao
```java 
@Component(value="userDao")
public class UserDao{
    
    public void add(){
        System.out.println("dao........");
    }
}
```
3、在 UserService类中，把UserDao作为一个属性，并在该属性上使用注解@Autowired自动装配，完成对象注入（根据类名UserDao找到该类并自动注入值）。
```java 
@Component(value="userService")
public class UserService{
    //（1）把UserDao对象作为类型属性
    //（2）在UserDao属性上使用注解完成对象注入,@Autowired自动装配
    @Autowired
    private UserDao userDao;

    public void add(){
        System.out.println("service........");
        //在service中得到Dao对象，才能调用Dao的方法
        userDao.add();
    }
}
```
也可以用@Resource(name="userDao")完成对象注入
```java 
@Component(value="userService")
public class UserService{
    //（1）把UserDao对象作为类型属性
    //（2）@Resource(name="userDao"),name的属性值为注解创建UserDao对象时的value值，
    // 这里创建UserDao对象的注解是@Component(value="userDao")
    @Resource(name="userDao")
    private UserDao userDao;

    public void add(){
        System.out.println("service........");
        //在service中得到Dao对象，才能调用Dao的方法
        userDao.add();
    }
}
```
### 配置文件和注解混合使用
1、创建对象操作使用配置文件方式实现
2、注入属性操作使用注解方式实现

1）创建UserService类
```java 
public class UserService{

    public void add(){
        System.out.println("service........");
        //在service中得到Dao对象，才能调用Dao的方法
        userDao.add();
    }
}
```
2、创建UserDao1类
```java 
public class UserDao{
    
    public void add(){
        System.out.println("dao1........");
    }
}
```
3、创建UserDao2类
```java 
public class UserDao{
    
    public void add(){
        System.out.println("dao2........");
    }
}
```
4、创建配置文件创建对象
```java
<!---开启注解扫描
    （1）到包里面扫描类、方法、属性上面是否有注解
--->
<context:component-scan base-package="com"></context:component-scan>

<!---配置对象--->
<bean id="userService" class="com.UserService"></bean>
<bean id="userDao1" class="com.UserDao1"></bean>
<bean id="userDao2" class="com.UserDao2"></bean>
```
5、在UserService中使用注解@Resource注入属性
```java 
public class UserService{
    @Resource(name="userDao1")
    private UserDao1 userDao1;
    
    @Resource(name="userDao2")
    private UserDao2 userDao2;
    
    public void add(){
        System.out.println("service........");
        //在service中得到Dao对象，才能调用Dao的方法
        userDao1.add();
        userDao2.add();
    }
}
```