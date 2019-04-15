title: spring-IOC
---

## spring概念
1、是开源的轻量级框架

#### 2、核心两部分
> 1）aop：面向切面编程，扩展功能不是修改源代码实现
2）ioc：控制反转，比如有一个类，类里有方法（不是静态方法），若要调用该类的方法，则需要先创建类的对象，通过对象来调用方法，而创建对象需要用new， 而控制反转则是对象的创建不通过new方式实现，而是交给spring框架配置创建类的对象，控制权不在开发者手中，而在上层框架中。

#### 3、一站式框架
spring在JavaEE三层结构中，每一层都提供不同的解决技术
web层：springMVC
service层：spring的IOC
dao层：spring的jdbcTemplate

> 注意：
1、单例类只能有一个实例。
2、单例类必须自己创建自己的唯一实例。
3、单例类必须给所有其他对象提供这一实例。

## IOC底层原理
1、底层原理使用技术
> * 1）xml配置文件
> * 2）dom4j解析xml
> * 3）工厂设计模式
> * 4）反射 

例：
```java
public class UserService{
}
```

```java
public class UserServlet{
    //得到UserService的对象
    //常规方法：new创建
    //IOC方法：工厂模式创建
    UserFactory. getService();
}
```

第一步：创建xml配置文件，配置要创建对象的类
```java
<bean id="userService" class="com.ct.UserService">
//class属性里要写全类名
```
第二步：创建工厂类，使用dom4j解析配置文件+反射
```java
public class UserFactory{
    //得到UserService对象的方法
    public static UserService getService(){
        //1、使用dom4j解析xml文件，根据id值userService得到class属性值
        String classValue="class属性值";
        //2、使用反射创建类对象
        Class clazz=Class.forName(classValue);
        //3、创建类对象
        UserService service=clazz.newInstance();
        return service;
    }
}
```
## spring的IOC操作
1、把对象的创建交给spring进行管理

2、IOC操作两部分
1）IOC的配置文件方式
2）IOC的注解方式

## spring的bean管理（xml方式）
bean实例化的三种方式
1）实用类的默认无参构造函数创建（重点）
若类中没有无参构造函数，则出现异常

2）使用静态工厂创建
创建静态方法，返回类对象
例：
```java
public class UserFactory{
    //得到UserService对象的静态方法
    public static UserService getService(){
        return new UserService();
    }
}
```
```java
<!---使用静态工厂创建对象--->
<bean id="userService" class="com.UserFactioy" factory-method="getService"></bean>
<!---class属性值为工厂类的全类名，factory-method的属性值为返回所要创建对象的静态方法名--->
```

3）使用实例工厂创建
创建不是静态方法，返回类对象
```java
public class UserFactory{
    //得到UserService对象的普通方法
    public  UserService getService(){
        return new UserService();
    }
}
```
```java
<!---使用实例工厂创建对象--->
<!---创建工厂对象--->
<bean id="userFactory" class="com.UserFactioy"></bean>
<bean id="userService" class="com.UserFactioy" factory-method="getService"></bean>
//class属性值为工厂类的全类名，factory-method的属性值为返回所要创建对象的方法名
```
## 属性注入
1、创建对象的时候，向类里面的属性设置属性值

2、Java中属性注入方式
> 1）set方法注入
```java
public class User{
    private String name;
    
    //setName方法为属性name注入值
    public void setName(String name){
        this.name=name;
    }
}
```

> 2）有参数构造方法注入
```java
public class User{
    private String name;
    
    //有参构造方法为属性name注入值
    public User(String name){
        this.name=name;
    }
}
```
> 3）接口注入
```java
//定义接口
public interface User{
    public void userName(String name);
}

//接口实现类
public class UserImpl implements User{
    private String name;
    
    //在实现类中实现方法并注入属性
    public void userName(String name){
        this.name=name;
    }
}
```
3、spring中属性注入方法
spring中只支持前两种属性注入方法
> 1）set方法注入（重点）
```java
<!---set方法注入--->
<bean id="user" class="com.User">
    <!---将value中的值注入到User类的属性name中--->
    <property name="name" value="张三李四" </property>
</bean>
```

> 2）有参数构造方法注入
```java
<!---有参数构造方法注入--->
<bean id="user" class="com.User">
    <!---将value中的值注入到User类的属性name中--->
    <constructor-arg name="name" value="张三李四" </constructor-arg>
</bean>
```

## 注入对象类型属性（重点）
例：
1、创建UserService和UserDao类
1）在UserService中得到UserDao对象
具体实现：
（1）在UserService里把UserDao对象作为类型属性
（2）创建生成UserDao对象属性的set方法
```java 
public class UserService{
    //（1）把UserDao对象作为类型属性
    private UserDao userDao;
    
    //（2）set方法
    public void setUserDao(UserDao userDao){
    //（1）在UserService里把UserDao对象作为类型属性
        this.userDao=userDao;
    }
    public void add(){
        System.out.println("service........");
        //在service中得到Dao对象，才能调用Dao的方法
        userDao.add();
    }
}
```
```java 
public class UserDao{
    
    public void add(){
        System.out.println("dao........");
    }
}
```
（3）在配置文件中配置注入关系
```java
<!---注入对象类型属性--->
<!---1、配置UserService和UserDao对象--->
<bean id="userDao" class="com.UserDao"></bean>
<bean id="userService" class="com.UserService">
    <!---注入UserDao对象
        name属性值：UserService类里的对象属性名称
        ref属性：UserDao配置的bean标签中的id值
        由ref属性代替value属性，因为这里配置的是对象，value的值为字符串
    --->
    <property name="userDao" ref="userDao" </property>
</bean>
```

### p名称空间注入
1、加一条schema约束
```java
xmlns:p="http://www.springframework.org/schema/p"
```
2、创建User类及其属性userName
```java
public class User{
    private String userName;
    
    //有参构造方法为属性name注入值
    public User(String userName){
        this.userName=userName;
    }
}
```
3、注入属性
```java
<bean id="user" class="com.User" p:userName="张三"></bean>
```
### spring注入复杂数据
1、数组
2、list集合
3、map集合
4、properties类型

第一步：编写测试类Person
```java
import java.util.List;
import java.util.Map;
import java.util.Properties;

public class Person {
    private String pname;

    public void setPname(String pname) {
        this.pname = pname;
    }

    private String[] arrs;
    private List<String> list;
    private Map<String,String> map;
    private Properties properties;

    public void setArrs(String[] arrs) {
        this.arrs = arrs;
    }
    public void setList(List<String> list) {
        this.list = list;
    }
    public void setMap(Map<String, String> map) {
        this.map = map;
    }
    public void setProperties(Properties properties) {
        this.properties = properties;
    }
}

```
第二步：配置bean
```java
<!---注入复杂类型属性值--->
<bean id="person" class="com.Person">
    <!---1、数组--->
    <property name="arrs">
        <!---name中为要注入的属性名称--->
        <list>
            <value>小王</value>
            <value>小李</value>
            <value>小马</value>
        </list>
    </property>

    <!---2、list--->
    <property name="list">
        <!---name中为要注入的属性名称--->
        <list>
            <value>小五</value>
            <value>小六</value>
            <value>小七</value>
        </list>
    </property>
    
    <!---3、map--->
    <property name="map">
        <!---name中为要注入的属性名称--->
        <map>
            <entry key="aa" value="小喵"></entry>
            <entry key="bb" value="小猪"></entry>
            <entry key="cc" value="小狗"></entry>
        </map>
    </property>
    
    <!---4、properties--->
    <property name="properties">
        <!---name中为要注入的属性名称--->
        <props>
            <prop key="driverclass">com.mysql.jdbc.Driver</prop>
            <prop key="username">root</prop>
        </props>
    </property>

</bean>
```

## IOC与DI的区别
1、IOC：控制反转，把对象创建交给spring进行配置（本质工作是创建对象）
例：在spring中利用bean配置来创建对象
```java
<!---创建User类的对象--->
<bean id="user" class="com.User">
   
</bean>
```

2、DI：依赖注入，向类里面的属性注入值（本质工作是为一个已经创建好的对象设置它的属性值）
例：
```java
<bean id="user" class="com.User">
    <!---为User类的对象中的username属性设置值为张三李四--->
    <property name="username" value="张三李四" </property>
</bean>
```

3、关系：依赖注入不能单独存在，需要在IOC基础之上完成操作（想要为一个对象设置它的属性值，必须要先创建对象）

## Spring整合web项目原理
例：如下
1、Person类
```java
public class Person {
    private String pname;

    public void setPname(String pname) {
        this.pname = pname;
    }

    public void test1(){
        System.out.println(pname+":test");
    }
}
```
2、bean配置
```java
<bean id="person" class="com.Person">
    <!---将value中的值注入到User类的属性name中--->
    <property name="pname" value="张三李四" </property>
</bean>
```
3、得到Person对象并调用其test1（）方法
```java
public class TestIOC{
    @Test
    public void testUser(){
        //1、加载spring配置文件，根据配置文件创建对象
        ApplicationContext context=new ClassPathXmlApplicationContext("bean.xml");

        //2、得到配置文件中创建的对象
        Person person=(Person)context.getBean("person");
        person.test1();
    }
}
```
整合原理：
1、加载spring核心配置文件
```java
        //1、加载spring配置文件，根据配置文件创建对象
        ApplicationContext context=new ClassPathXmlApplicationContext("bean.xml");
```
ApplicationContext是多实例的，每次new ClassPathXmlApplicationContext()这个对象将会使得效率很低，因为每创建一个对象就会调用一次。

2、改善思想：把加载配置文件和创建对象过程，在服务器启动时完成，把压力交给服务器。

3、实现原理：
1）ServletContext对象
2）监听器

（1）在服务器启动的时候，为每个项目创建一个ServletContext对象
（2）在ServletContext对象创建的时候，使用监听器可以知道ServletContext对象什么时候创建
（3）当监听器知道ServletContext对象创建时，就加载spring配置文件，把配置文件配置的对象创建出来
（4）把创建出来的对象放到ServletContext域对象里面（setAttribute方法可以实现该功能）
（5）获取对象时，到ServletContext域中得到（getAttribute方法得到）