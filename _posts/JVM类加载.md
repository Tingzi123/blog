title: JVM类加载
---
## JVM类加载
1、Java中，类型的加载、连接与初始化过程都是在程序运行期间完成的。
1）加载：查找并加载类的二进制数据（将类从磁盘文件写入内存）
2）连接：
（1）验证：确保被加载的类的正确性
（2）准备：为类的静态变量分配内存，并将其初始化为默认值（例如，静态的整型值初始化为默认值为0，由Java虚拟机来完成）
（3）解析：把类中的符号引用转换为直接引用
3）初始化：为类的静态变量赋正确的初始值（程序员主观初始化赋值，例int a=1）
4）使用
5）卸载：从内存中销毁类

2、类加载器：加载类，将类写入内存

3、jvm结束生命周期的几种方式：
1）执行System.exit()方法
2）程序正常执行结束
3）程序执行过程中遇到异常或错误而异常终止
4）操作系统的错误

4、Java程序对类的使用方式：
1）主动使用
（1）创建类的实例
（2）访问某个类或接口的静态变量（取值getStatic()），或对该静态变量赋值(putStatic()）
（3）调用类的静态方法(invokeStatic())
（4）反射（如Class.forName("com.Test")）
（5）初始化一个类的子类（初始化子类时，会先去初始化父类，这是一个对父类的主动使用）
（6）Java虚拟机启动时被标明为启动类的类（Java Test，包含main（）方法的类）
（7）JDK1.7开始提供的动态语言支持：
java.lang.invoke.MethodHandle实例的解析结果REF_getStatic,REF_putStatic,REF_invokeStatic句柄对应的类没有初始化，则初始化。

2）被动使用
除以上七种情况外，其他使用Java类的方法都被看做是对类的被动使用，都不会导致类的初始化

注：所有的Java虚拟机实现必须在每个类或接口被Java程序“首次主动使用”时才初始化他们
举个很有用栗子：一个父类Parent，一个子类Child
```java
package jvm;
//-XX:TraceClassLoading:用于追踪类的信息并打印出来
public class ClassLoad {

	public static void main(String[] args) {
	
		//用子类名调用父类的静态属性str
		System.out.println(Child.str);
	}
}

class Parent{
    //静态属性str
	public static String str="Hello world";
	
	//静态代码块
	static {
		System.out.println("Parent!!!");
	}
}

class Child extends Parent{
	//静态代码块
	static {
		System.out.println("Child!!!");
	}
}
```
输出：
```java
Parent!!!
Hello world
//用子类名调用父类的静态属性str,输出的是父类的静态代码块和父类的静态属性值
//因为调用父类的静态属性str，符合Java程序对类的主动使用第二点（访问某个类或接口的静态变量（取值getStatic()），或对该静态变量赋值(putStatic()）），则Child.str的过程就是对父类的主动使用，但是没有主动使用子类，所以Java虚拟机不会初始化子类，则不会执行，虽然子类没有初始化，但是依然加载了，可用TraceClassLoading助记符来追踪的知。
```
再看下面的代码：
```java
package jvm;
//-XX:TraceClassLoading:用于追踪类的信息并打印出来
public class ClassLoad {

	public static void main(String[] args) {
	
		//用子类名调用子类的静态属性str2
		System.out.println(Child.str2);
	}
}

class Parent{
//对于静态字段来说，只有直接定义了该字段的类才会被初始化
    //静态属性str
	public static String str="Hello world";
	
	//静态代码块
	static {
		System.out.println("Parent!!!");
	}
}

class Child extends Parent{
//对于静态字段来说，只有直接定义了该字段的类才会被初始化
//对一个子类初始化时，要求其父类已经全部初始化
    //子类静态属性str2
    public static String str2="Hello world";
	//静态代码块
	static {
		System.out.println("Child!!!");
	}
}
```
输出：
```java
Parent!!!
Child!!!
Hello world
//用子类名调用子类的静态属性str2,输出的是父类的静态代码块、子类的静态代码块和子类的静态属性值
//因为调用子类的静态属性str，符合Java程序对类的主动使用第二点（访问某个类或接口的静态变量（取值getStatic()），或对该静态变量赋值(putStatic()）），则Child.str22的过程就是对子类的主动使用，又因为子类继承与父类，这也是一种对父类的主动使用，所以父类与子类都会初始化并执行。
```
### 类的加载
是指将类的.class文件中的二进制数据读入内存，将其放在运行时数据区的方法区内，然后在内存中创建一个java.lang.Class对象,用来封装类在方法区内的数据结构

1、加载.class文件的方式
1）从本地系统中直接加载
2）通过网络下载.class文件
3）从zip,jar等归档文件中加载
4）从专有数据库加载
5）将Java源文件动态编译为.class文件 


### 常量
编译阶段，常量会被存入调用这个常量的那个方法所在类的常量池中，本质上，调用类并没有直接引用到定义常量的类，因此并不会触发定义常量的类的初始化。之后定义常量的类与调用常量的类就没有任何关系了，删除定义常量的class文件也不影响运行。
例：
```java
package jvm;

public class ChangLiang {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(Test.str);
	}
}

class Test{
	public static String str="hello world";
	
	static {
		System.out.println("Test类");
	}
}
```
输出：
```java
Test类
hello world
```
再看下列代码：
```java
package jvm;

public class ChangLiang {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(Test.str);
	}
}

class Test{
    //静态常量属性str
	public static final String str="hello world";
	
	static {
		System.out.println("Test类");
	}
}

```
输出：
```java
hello world
//因为str是静态常量，在编译阶段，这个常量会被存入调用这个常量的那个方法所在类的常量池中，即str会被放在ChangLiang类的常量池中，本质上，调用类并没有直接引用到定义常量的类（Test类），因此并不会触发定义常量的类的初始化。之后定义常量的类（Test）与调用常量的类（ChangLiang）就没有任何关系了，删除定义常量（Test)的class文件也不影响运行。对调用常量的类ChangLiang反编译，会发现Test.str这一句代码直接被常量"hello world"替换。
```
### 编译期常量与运行期常量的区别
上面的例子是编译期常量，下面演示运行期常量
```java
package jvm;

import java.util.UUID;

public class MyTest {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		System.out.println(MyParent.str);
	}

}
class MyParent{
	//UUID 是 通用唯一识别码（Universally Unique Identifier）的缩写，是一种软件建构的标准
	//UUID.randomUUID().toString()是随机的，编译器无法确定
	public static final String str=UUID.randomUUID().toString();
	
	static {
		System.out.println("UUID CODE");
	}
}
```
输出：
```java
UUID CODE
f624dd9f-5d47-42f4-8fb2-41812b231bda
```
注意：当一个常量并非编译器可以确定，那么其值就不会被放到调用类的常量池中，这时程序运行时，会会主动使用定义这个常量所在的类，则会导致该类被初始化（在这个例子中，对于public static final String str=UUID.randomUUID().toString()，是随机的，在编译器无法确定，则会加载它所在的类MyParent,从而输出UUID CODE）


### 数组创建本质
对于数组类型，它的类加载器是由Java虚拟机在运行期动态创建的，Java虚拟机返回的类加载器类型与数组中元素的类加载器类型一样（例如一个string类型的数组，String的类加载器是根类加载器，那么Java虚拟机为数组类型创建的类加载器也是根类加载器，），如果元素类型为原生类型，则没有类加载器（例如int类型）。
一个前言
```java
package jvm;

public class ArrayClass {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		TestArray a=new TestArray();//首次实例化
		TestArray b=new TestArray();//再次实例化
	}

}

class TestArray{
	static {
		System.out.println("TestArray.........");
	}
}
```
输出
```java
TestArray.........
```
结果分析：当对一个类首次实例化时，会主动调用并初始化，再次则不会主动使用了，所以实例化两次，只打印一次。

正文：
```java
package jvm;

public class ArrayClass {

	public static void main(String[] args) {
		TestArray[] a=new TestArray[1];
	}

}

class TestArray{
	static {
		System.out.println("TestArray.........");
	}
}


```
输出:
上面这个代码不会有任何输出，因为数组的类型实际上并不是TestArray。可以用getClass()方法查看数组的类型，如下：
```java
package jvm;

public class ArrayClass {

	public static void main(String[] args) {
		TestArray[] a=new TestArray[1];
		//查看数组所属类型
		System.out.println(a.getClass());
	}
}

class TestArray{
	static {
		System.out.println("TestArray.........");
	}
}
```
输出：
```java
class [Ljvm.TestArray;
```
由输出可知，数组的类型是“[Ljvm.TestArray;”，这个类型由Java虚拟机在运行期动态创建。

下面再看看二维数组：
```java
package jvm;

public class ArrayClass {

	public static void main(String[] args) {
		TestArray[][] a=new TestArray[1][1];
		System.out.println(a.getClass());
	}
}

class TestArray{
	static {
		System.out.println("TestArray.........");
	}
}
```
输出：
```java
class [[Ljvm.TestArray;
```
由输出知，二位数组的类型为“[[Ljvm.TestArray;”，比一维数组多一个方括号“[”。

下面调用getSuperClass()方法查看一下这个数组类型的父类型
```java
package jvm;

public class ArrayClass {

	public static void main(String[] args) {
		TestArray[][] a=new TestArray[1][1];
		//查看数组类型
		System.out.println(a.getClass());
		//查看数组类型的父类型
		System.out.println(a.getClass().getSuperclass());
	}
}

class TestArray{
	static {
		System.out.println("TestArray.........");
	}
}
```
输出：
```java
class [[Ljvm.TestArray;
class java.lang.Object
```
由输出知，二位数组的类型为“[[Ljvm.TestArray;”，二位数组类型的父类型为“java.lang.Object”，经实验，一维数组的父类型也是“java.lang.Object”

引用类型与基本数组类型的区别：
```java
package jvm;

public class ArrayClass {

	public static void main(String[] args) {
		TestArray[] a=new TestArray[1];
		//查看引用（数组）类型
		System.out.println(a.getClass());
		//查看引用（数组）类型的父类型
		System.out.println(a.getClass().getSuperclass());
		System.out.println();
		
		int[] ints=new int[1];
		//查看基本类型（int)的数组类型
		System.out.println(ints.getClass());
		//查看基本类型（int)的数组类型的父类型
		System.out.println(ints.getClass().getSuperclass());
		System.out.println();
		
		char[] chars=new char[1];
		//查看基本类型（int)的数组类型
		System.out.println(chars.getClass());
		//查看基本类型（int)的数组类型的父类型
		System.out.println(chars.getClass().getSuperclass());
	}
}

class TestArray{
	static {
		System.out.println("TestArray.........");
	}
}
```
输出：
```java
class [Ljvm.TestArray;
class java.lang.Object

class [I
class java.lang.Object

class [C
class java.lang.Object
```
由输出可知：对于数组实例来说，其类型是jVM再运行期间动态创建的，表示为“[Ljvm.TestArray;”这种格式，对于基本类型的数组来说，如int类型表示为“[I”。父类都是object。
对于数组来说：javaDOC经常将构成数组的元素为Component，实际上就是将数组降低一个维度。

### 助记符
-XX:+<option>表示开启option选项
-XX:-<option>表示关闭option选项
-XX:<option>=<value>表示将option选项的值置为value

TraceClassLoading:用于追踪类的信息并打印出来
ldc:表示将int、float或string类型二等常量从常量池中推送至栈顶
blpush:将单字节（-128至127）二等常量推送至栈顶
sipush:将短整型常量值（-32768至32767）二等常量推送至栈顶
iconst_1:将int型的1推送至栈顶（iconst_-1 - iconst_5共七个数）
anewarray:创建一个引用类型的（如类、接口、数组）数组，并将其引用值压入栈顶
newarray:创建一个指定额原始类型（如int、float、char）等，，并将其引用值压入栈顶