title: Spring-AOP
---
## AOP概念
1、AOP:面向切面（方面）编程，扩展功能不修改源代码实现
2、AOP采取横向抽取机制，取代了传统纵向继承体系重复性代码（性能监视、事务管理、安全检查、缓存）
## AOP原理
1、创建User类
```java
public class User{
    //添加数据的用户方法
    public void add(){
        //添加逻辑
        //若要添加日志功能，需在此修改源代码添加日志逻辑
    }
}
```
2、纵向抽取机制（使用继承实现来增强功能）
```java
public class baseUser extends User{
    //添加数据的用户方法
    public void add(){
        //添加逻辑
        //功能扩展，添加日志功能
        //调用父类方式实现日志功能
        super.add();
    }
}
```
缺点：若父类的方法名称发生改变，在子类调用的方法也要改变

3、横向抽取机制（AOP来增强功能）
底层使用动态代理实现：

第一种情况：使用jdk动态代理，针对有接口的情况
1）创建接口Dao
```java
public class Dao{
    public void add();
}
```
2）创建接口的实现类
```java
public class DaoImpl implements Dao{
    public void add(){
        //添加逻辑
    }
}
```
3）使用动态代理方式，创建接口实现类的代理对象
注：这里具体指创建和DaoImpl类的平级对象，这个对象不是真正的对象，但是代理对象可以实现和DaoImpl相同的功能，需要扩展功能时，通过代理对象来扩展。

第二种情况：使用cglib动态代理实现：针对没有接口的情况
1）、创建User类
```java
public class User{
    //添加数据的用户方法
    public void add(){
        //若要增强功能，使用动态代理
    }
}
```
2）cglib动态代理实现：创建User的子类的代理对象，在子类里调用父类的方法来完成功能增强。

### AOP相关术语
1、Joinpoint（连接点）：指那些被拦截到的点。spring中，这些点指的是方法，因为spring只支持方法类型的连接点。
2、Pointcut（切入点）：指我们要对哪些Joinpoint（连接点）进行拦截的定义
3、Advice（通知/增强）：通知是指拦截到JoinPoint之后要做的事情，分为前置通知、后置通知、异常通知、最终通知、环绕通知（切面要完成的功能）
4、Aspect（切面）：是切入点和通知的结合
5、Introduction（引介）：引介是一种特殊的通知在不修改代码的前提下，引介可以在运行期为类动态地添加一些方法或Field
6、Target（目标对象）：代理的目标对象（要增强的类）
7、Weaving（织入）：把增强应用到目标的过程
8、Proxy（代理）：一个类被AOP织入增强后，就产生一个结果代理类

例：有一个User类
```java
public class User{
    public void add(){}
    public void update(){}
    public void delete(){}
    public void findAll(){}
}
```
1）连接点：类里面那些可以被增强的方法，例如User类中的add()、update()、delete()、findAll()方法。
2）切入点：类面边实际被增强的方法，例如User类中只有add()和update()方法被增强，则add()和update()方法被称为切入点。
3）通知/增强：实现功能增强的实际逻辑，例如扩展日志功能，这个功能则称为增强。
4）切面：把增强应用到具体的方法上面的过程称为切面（把增强用到切入点的过程）
5）引介：可以动态向类中添加属性和方法
6）目标对象：增强方法所在的类，例如要增强add()方法。add()方法所在的类是User，User则称为目标对象。
7）织入：把增强用到类的过程
8）代理：一个类被AOP织入增强后，就产生一个结果代理类

通知的几种类型：
1）前置通知：在方法之前执行
2）后置通知：在方法之后执行
3）异常通知：在方法出现异常时执行
4）最终通知：在后置之后执行（类似于try{}catch{}finall{})
5）环绕通知：在方法之前和之后执行

## Spring的AOP操作
1、在spring里面进行AOP操作，使用Aspectj实现
（1）Aspectj是一个面向切面的框架，基于Java语言，Aspectj2.0以后新增了对Aspectj切点表达式的支持。
（2）Aspectj不是spring的一部分，和spring一起使用进行AOP操作

2、使用Aspectj实现AOP的两种方式
（1）基于Aspectj的xml配置文件方式
（2）基于Aspectj的注解方式

3、需额外导入spring-aop约束

### 基于Aspectj的xml配置文件方式
1、创建Book类
```java
package aop;

public class Book{
    public void add(){
        System.out.println("add.....");
    }
}
```
2、创建增强类MyBook
```java
package aop;

public class MyBook {
	public void before1() {
		System.out.println("前置增强。。。。。。");
	}
	
	public void after1() {
		System.out.println("后置增强。。。。。。");
	}
	
	public void around1(ProceedingJoinPoint proceedingJoinPoint) {
		//方法之前环绕
		System.out.println("方法之前环绕增强。。。。。。");
		
		//执行被增强的方法
		proceedingJoinPoint.proceed();
		
		//方法之前后
		System.out.println("方法之后环绕增强。。。。。。");
	}
}
```
3、使用表达式来配置切入点
1）切入点：实际增强的方法
2）常用的表达式：
```java
execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)
```
几种常用表达式写法：
（1）execution(* aop.Book.add(..))
（2）execution(* aop.Book.*(..))
（3）execution(* *.*(..))
（4）execution(* *.add*(..)),匹配所有以add开头的方法

4、配置文件
```java
<!---1、配置对象--->
<bean id="book" class="aop.Book"></bean>
<bean id="myBook" class="aop.MyBook"></bean>

<!---2、配置aop操作--->
<aop:config>
    <!---1）配置切入点--->
    <aop:pointcut expression="execution(* aop.Book.add(..))" id="pointcut1"/>
    
    <!---2）配置切面--->
    <aop:aspect ref="myBook">
        <!---配置增强类型
            method：增强类里面实际要用来做增强的方法
        --->
        
        <!---（1）前置增强--->
         <aop:before method="before1" pointcut-ref="pointcut1"/>
     
        <!---（2）后置增强--->
        <aop:after-returning method="after11" pointcut-ref="pointcut1"/>
         
        <!---（3）环绕增强--->
        <aop:around method="around1" pointcut-ref="pointcut1"/>
        
    </aop:aspect>
</aop:config>
```
5、测试
```java
package aop;

public class AopTest {
	@Test
	public void testService() {
		ApplicationContext context=new ClassPathXmlApplicationContext("bean.xml");
		Book book=context.getBean("book");
		book.add();
	}
}

```
### 基于Aspectj的注解方式
1、创建Book类
```java
package aop;

public class Book{
    public void add(){
        System.out.println("add.....");
    }
}
```
2、创建增强类MyBook
```java
package aop;

public class MyBook {
	public void before1() {
		System.out.println("前置增强。。。。。。");
	}
	
	public void after1() {
		System.out.println("后置增强。。。。。。");
	}
	
	public void around1(ProceedingJoinPoint proceedingJoinPoint) {
		//方法之前环绕
		System.out.println("方法之前环绕增强。。。。。。");
		
		//执行被增强的方法
		proceedingJoinPoint.proceed();
		
		//方法之前后
		System.out.println("方法之后环绕增强。。。。。。");
	}
}
```
3、配置文件
创建对象并开启AOP操作
```java
<!---1、开启aop操作--->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>

<!---2、配置对象--->
<bean id="book" class="aop.Book"></bean>
<bean id="myBook" class="aop.MyBook"></bean>
```
4、在增强类MyBook上使用注解完成AOP操作
```java
package aop;

//1）在类上加@Aspect
@Aspect
public class MyBook {
    //2、在方法上加具体的注解
    @Before(value="execution(* aop.Book.add(..))")
	public void before1() {
		System.out.println("前置增强。。。。。。");
	}
	
	@AfterRuturning(value="execution(* aop.Book.add(..))")
	public void after1() {
		System.out.println("后置增强。。。。。。");
	}
	
	@Around(value="execution(* aop.Book.add(..))")
	public void around1(ProceedingJoinPoint proceedingJoinPoint) {
		//方法之前环绕
		System.out.println("方法之前环绕增强。。。。。。");
		
		//执行被增强的方法
		proceedingJoinPoint.proceed();
		
		//方法之前后
		System.out.println("方法之后环绕增强。。。。。。");
	}
}
```
5、测试
```java
package aop;

public class AopTest {
	@Test
	public void testService() {
		ApplicationContext context=new ClassPathXmlApplicationContext("bean.xml");
		Book book=context.getBean("book");
		book.add();
	}
}

```