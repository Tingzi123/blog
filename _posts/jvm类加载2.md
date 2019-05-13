title: JVM类加载2
---
## JVM接口初始化规则
1、当一个接口再初始化时，并不要求其父接口都完成初始化，只有当真正使用到父接口时（如引用接口中定义的常量时）才会初始化。（对于类来说，子类初始化之前，它的父类要先全部初始化）
2、当一个类初始化时，并不要求先初始化其所实现的接口
注：1）只有当程序首次使用特定接口的静态变量时，才会导致该接口的初始化。
2）只有当程序访问的静态变量或静态方法确实在当前类或接口中定义时，才可以认为是对类或接口的主动使用
```java
package jvm;

public class InterFaceClass {

	public static void main(String[] args) {
		System.out.println(Childs.b);
	}
}

interface Parents{
//接口中的属性就是static和final，这里显示定义为了更加清楚
	public static int a=5;
}

interface Childs extends Parents{
	public static final int b=6;
}
```
输出：
```java
6
```
在上面这个例子中，
Childs.b调用子接口的属性b，属性b的定义为public static final int b=6，这是一个常量，在编译期间会被放入调用它的类的常量池中，这时，不会初始化Childs类，虽然子接口继承自父接口，但没有使用到父接口，所以没有主动使用父接口，不会初始化父接口，将子接口的class文件删除也不影响运行。（子接口初始化，父接口不一定初始化）

再看下面的例子：
```java
package jvm;

import java.util.Random;

public class InterFaceClass {

	public static void main(String[] args) {
		System.out.println(Childs.b);
	}
}

interface Parents{
	public int a=5;
}

interface Childs extends Parents{
	public static final int b=new Random().nextInt();
}
```
输出：
```java
2024458112
```
在上面的例子中，Childs.b调用子接口的属性b,但b的定义为public static final int b=new Random().nextInt()，是一个随机数，要在运行期间才能确定，所以这个值不会被放到调用它的类的常量池中，这时会主动使用父接口，使父接口初始化，若把父接口的class文件删除，会报找不到对象异常。

上面的例子不是很准确的说明接口的初始化的规则，但是我太懒了，并且上面的例子对于Final关键字有更深的理解，所以我懒得去改了，请看下面这个比较确切的例子：
### 验证接口初始化规则第二点：
当一个实现接口的类在初始化时，并不要求其接口初始化
1、接口
```java
package jvm;

public class InterfaceInit {

	public static void main(String[] args) {
	    //调用MyChilds的静态属性b
		System.out.println(MyChilds.b);
		
	}
}

interface MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyGrandpa...........");
		}
	};
}

interface MyParents extends MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyParents...........");
		}
	};
}

class MyChilds implements MyParents{
	public static int b=5;
}
```
输出：
```java
5
```
由输出可知：当初始化一个接口的实现类时，并不一定会初始化这个接口（接口初始化规则第二条）
1）MyChilds.b，在InterfaceInit类中调用MyChilds的静态属性b，主动使用MyChilds类并初始化它。
2）但是并没有主动使用它的接口MyParents，所以不会初始化MyParents类，也不会初始化MyParents的父类MyGrandpa

2、把接口改成类
 ```java
package jvm;

public class InterfaceInit {

	public static void main(String[] args) {
		System.out.println(MyChilds.b);
		
	}
}

class MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyGrandpa...........");
		}
	};
}

class MyParents extends MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyParents...........");
		}
	};
}

class MyChilds extends MyParents{
	public static int b=5;
}
```
输出：
```java
MyGrandpa...........
MyParents...........
5
```
由输出知：当初始化一个子类时，一定会先初始化它的父类
1）MyChilds.b，在InterfaceInit类中调用MyChilds的静态属性b，主动使用MyChilds类并初始化它。
2）由于MyChilds继承自MyParents类，所以会主动使用MyParents类，并初始化MyParents类
3）初始化MyParents类之前，由MyParents类继承自MyParents的父类MyGrandpa类，所以会主动使用MyGrandpa类，并且会初始化MyGrandpa类
4）在MyGrandpa类初始化阶段，它的静态属性初始化代码块中会打印"MyGrandpa..........."
5）MyGrandpa类初始化完之后，是MyParents类初始化，在MyParents类初始化阶段，它的静态属性初始化代码块中会打印"MyParents..........."

3、把定义的静态变量改成final static变量
 ```java
package jvm;

public class InterfaceInit {

	public static void main(String[] args) {
		System.out.println(MyChilds.b);
		
	}
}

class MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyGrandpa...........");
		}
	};
}

class MyParents extends MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyParents...........");
		}
	};
}

class MyChilds extends MyParents{
//静态常量属性
	public final static int b=5;
}
```
输出：
```java
5
```
由输出可知：
1）MyChilds.b，在InterfaceInit类中调用MyChilds的静态常量属性b，主动使用MyChilds类
2）但是b是静态常量，在编译阶段将b放入它的调用类InterfaceInit的常量池中，之后属性b的定义类MyChilds与它的调用类InterfaceInit就没有关系了，不会初始化MyChilds，就不会主动使用MyChilds的父类MyParents且不会初始化。

### 验证接口初始化规则第一点：
当一个接口在初始化时，并不要求其父接口都完成初始化
例子：
 ```java
package jvm;

public class InterfaceInit {

	public static void main(String[] args) {
		System.out.println(MyParents.thread);
		
	}
}

interface MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyGrandpa...........");
		}
	};
}

interface MyParents extends MyGrandpa{
	public static Thread thread=new Thread(){
		//一个实例化代码块
		{
			System.out.println("MyParents...........");
		}
	};
}
```
输出：
```java
MyParents...........
Thread[Thread-0,5,main]//Thread的toString()方法
```
由输出可知：
1）MyParents.thread，在InterfaceInit类中调用接口MyParents的静态属性thread，主动使用MyParents接口并初始化它的静态属性thread，在静态属性thread的初始化代码块中，打印"MyParents..........."
2）虽然接口MyParents继承自接口MyGrandpa，但是并没有初始化父接口MyGrandpa


### 类加载的准备阶段与初始化阶段
类加载过程：
```java
1）加载：查找并加载类的二进制数据（将类从磁盘文件写入内存）
2）连接：
（1）验证：确保被加载的类的正确性
（2）准备：为类的静态变量分配内存，并将其初始化为默认值（例如，静态的整型值初始化为默认值为0，由Java虚拟机来完成）
（3）解析：把类中的符号引用转换为直接引用
3）初始化：为类的静态变量赋正确的初始值（程序员主观初始化赋值，例int a=1）
4）使用
5）卸载：从内存中销毁类
```
例子：
 ```java
 package jvm;

public class InitClass {

	public static void main(String[] args) {
	    //调用Singleton的静态方法来创建实例singleton
		Singleton singleton=Singleton.getInstance();
		System.out.println(singleton.count1);
		System.out.println(singleton.count2);
	}
}

class Singleton{
	public static int count1;
	public static int count2=0;
	
	private static Singleton singleton=new Singleton();
	
	private Singleton(){
		count1++;
		count2++;
	}
	
	public static Singleton getInstance() {
		return singleton;
	}
}
```
输出：
```java
1
1
```
由输出知：
1）Singleton.getInstance()调用Singleton的静态方法来创建实例singleton，所以会主动使用类Singleton，则先加载类，然后连接
2）在连接的第二阶段准备这个过程中，Java虚拟机为类Singleton的静态变量count1,count2,singleton分配内存，并将其初始化为默认值:count1=0,count2=0,singleton=null
3）连接完了以后是初始化过程，执行代码中显示赋初值的部分，public static int count2=0，private static Singleton singleton=new Singleton()
4）new Singleton()会执行构造函数，在构造函数中执行	count1++，count2++。
5）所以最后输出count1的值为1，count2的值为1。

例子：
 ```java
 package jvm;

public class InitClass {


	public static void main(String[] args) {
		Singleton singleton=Singleton.getInstance();
		System.out.println("main函数中的值");
		System.out.println(singleton.count1);
		System.out.println(singleton.count2);
	}
}

class Singleton{
	public static int count1;
	
	private static Singleton singleton=new Singleton();
	
	private Singleton(){
		count1++;
		count2++;
		
		System.out.println("构造函数内的值");
		System.out.println(singleton.count1);
		System.out.println(singleton.count2);
	}
	
	public static int count2=0;
	
	public static Singleton getInstance() {
		return singleton;
	}
}
```
输出：
```java
构造函数内的值
1
1
main函数中的值
1
0
```
由输出知：
1）Singleton.getInstance()调用Singleton的静态方法来创建实例singleton，所以会主动使用类Singleton，则先加载类，然后连接
2）在连接的第二阶段准备这个过程中，Java虚拟机为类Singleton的静态变量count1,count2,singleton分配内存，并将其初始化为默认值:count1=0,count2=0,singleton=null
3）连接完了以后是初始化过程，执行代码中显示赋初值的部分，private static Singleton singleton=new Singleton()
4）new Singleton()会执行构造函数，在构造函数中执行	count1++，count2++，构造函数中执行完两个语句后，count1的值为1，count2的值为1。
5）构造函数为属性singleton初始化之后，下一条初始化语句是public static int count2=0，这时将count2的值初始化为0
6）所以最后输出count1=1，count2=0


### 初始化进一步理解
---
```java
 package jvm;

public class FinalClass {
	static {
		System.out.println("FinalClass-------");
	}
	public static void main(String[] args) {
		System.out.println(ChildTest.b);

	}

}
class ParentTest{
	static int a=3;
	
	static {
		System.out.println("ParentTest--------");
	}
}
class ChildTest extends ParentTest{
	static int b=4;
	
	static {
		System.out.println("ChildTest--------");
	}
}
```
输出：
```java
FinalClass------
ParentTest--------
ChildTest--------
4
```
注：static代码块也被看做是要初始化的变量，同static变量一样
有输出知：
1）由主动使用的条件第六条（Java虚拟机启动时被标明为启动类的类（Java Test，包含main（）方法的类））知，含有main方法的类为启动类FinalClass，会主动使用并初始化，初始化阶段会执行static代码块打印"FinalClass-------"
2）初始化FinalClass类之后，执行main方法中的ChildTest.b代码
3）ChildTest.b调用ChildTest类的静态变量b，所以会主动使用ChildTest并初始化
4）在初始化ChildTest类之前会先初始化它的父类ParentTest，在父类ParentTest初始化时打印"ParentTest--------"
5）父类ParentTest初始化完之后，初始化ChildTest打印"ChildTest--------"
6）初始化ChildTest类完成之后，返回ChildTest类属性b的值到它的调用处main方法中并打印。

### 首次使用与再次使用类
类仅会被初始化一次，在首次主动使用它时初始化。
例子:
```java
 package jvm;

public class FinalClass {
	static {
		System.out.println("FinalClass-------");
	}
	public static void main(String[] args) {
		System.out.println("----------");
		System.out.println();
		
		ParentTest p;
		System.out.println("----------");
		System.out.println();
		
		ParentTest pa=new ParentTest();
		System.out.println("----------");
		System.out.println();
		
		System.out.println(pa.a);
		System.out.println("----------");
		System.out.println();
		
		System.out.println(ChildTest.b);
		System.out.println("----------");
		System.out.println();
	}
}
class ParentTest{
	static int a=3;
	
	static {
		System.out.println("ParentTest--------");
	}
}
class ChildTest extends ParentTest{
	static int b=4;
	
	static {
		System.out.println("ChildTest--------");
	}
}
```
输出：
```java
FinalClass-------
----------

----------

ParentTest--------
----------

3
----------

ChildTest--------
4
----------
```
由输出：
1）由主动使用的条件第六条（Java虚拟机启动时被标明为启动类的类（Java Test，包含main（）方法的类））知，含有main方法的类为启动类FinalClass，会主动使用并初始化，初始化阶段会执行static代码块打印"FinalClass-------"
2）对于”ParentTest p;“，并没有实例化ParentTest，所以没有主动使用ParentTest，也不会初始化，所以不会打印
3）对于”ParentTest pa=new ParentTest()“，是对ParentTest类的首次实例化，所以会初始化并打印其静态代码块中的内容”ParentTest--------“（ParentTest没有父类，所以直接初始化本身）
4）对于”pa.a“，是对”ParentTest pa=new ParentTest()“的首次实例化的结果，直接打印ParentTest的属性a。
5）对于”ChildTest.b“，调用ChildTest的静态属性b,会主动首次使用ChildTest类并对它初始化，在他初始化之前，应先初始化它的父类ParentTest，但在这里，它的父类已经初始化完成，不必再次初始化，所以不会再次打印父类静态代码块的内容，直接初始化ChildTest自身并打印”ChildTest--------”，然后打印”ChildTest.b“的结果。

### 静态变量（静态代码块也可看做静态变量）定义在哪个类，才会对哪个类主动使用并初始化 
例子：
```java
package jvm;

public class FirstClassLoad {

	public static void main(String[] args) {
		System.out.println(Child1.a);
		System.out.println();
		
		Child1.doSomething();
	}
}
class Parents1{
	static int a=3;
	
	static {
		System.out.println("Parents1!!!");
	}
	
	static void doSomething() {
		System.out.println("doSomething!!!");
	}
}

class Child1 extends Parents1{
	static {
		System.out.println("Child1!!!");
	}
}
```
输出：
```java
Parents1!!!
3

doSomething!!!
```
由输出知：
1）在main方法中有Child1.a，通过子类Child1调用父类的静态属性a,这时，静态变量定义在哪个类才会对哪个类初始化，所以不会初始化子类Child1，而是初始化父类Parents1，并打印“Parents1!!!”，再打印Child1.a的结果3
2）对于	Child1.doSomething()，doSomething()方法同样定义在父类，会主动使用父类并初始化，但是父类已经初始化过了，所以直接打印	Child1.doSomething()的结果。

### 反射导致一个类的初始化
加载类不是对类的主动使用，不会导致类的初始化，反射是对类的主动使用，会导致类的初始化。
例子：
```java
package jvm;

public class ClassForName {

	public static void main(String[] args) throws ClassNotFoundException {
		//获取一个类加载器
		ClassLoader loader=ClassLoader.getSystemClassLoader();
		//加载jvm.ClassLoadTest1
		Class<?> clazz=loader.loadClass("jvm.ClassLoadTest1");
		System.out.println(clazz);
		System.out.println();
		
		clazz=Class.forName("jvm.ClassLoadTest1");//反射
		System.out.println(clazz);
	}
}

class ClassLoadTest1{
	static {
		System.out.println("ClassLoadTest!!!");
	}
}
```
输出：
```java
class jvm.ClassLoadTest1

ClassLoadTest!!!
class jvm.ClassLoadTest1
```
由输出可知：
1）调用ClassLoader类的loadClass（）方法加载一个类，不是对类的主动使用，并不会导致类的初始化，所以不会初始化ClassLoadTest1类，不打印这个类中的静态代码块。
2）对于clazz=Class.forName("jvm.ClassLoadTest1")是一种反射，是对类ClassLoadTest1的主动使用，会导致类ClassLoadTest1的初始化，所以会打印静态代码块。