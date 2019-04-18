title: jvm类加载3
---
### Java类加载过程（详细）
1、加载：查找并加载类的二进制数据（读入虚拟机）
2、连接：
（1）验证：确保被加载的类的正确性
（2）准备：Java虚拟机为类的静态变量分配内存，并将其初始化为默认值（例如，静态的整型值初始化为默认值为0，但是第三个阶段初始化阶段由程序员主观显示的赋值才是真正的初始化，这里Java虚拟机事先设置默认初值只是为了防止空异常）
（3）解析：在类型的常量池中寻找类、接口、字段和方法的符号引用，把这些符号引用替换成直接引用。
3、初始化：为类的静态变量赋正确的初始值（程序员主观初始化赋值，例static int a=1）
4、类实例化：
（1）为新的对象分配内存
（2）为实例变量赋默认值
（3）为实例变量赋正确的初始值
（4）Java编译器为它编译的每一个类文件都至少生成一个实例初始化方法，在Java的class文件中，这个实例初始化方法被称为“<init>”。针对源代码中的每一个类的构造方法，Java编译器都产生一个“<init>”方法。
5、卸载：从内存中销毁类

### 类的加载
1、类的加载的最终产品是位于内存中的class对象
2、class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区的数据结构的接口

3、有两种类型的类加载器
1）Java虚拟机自带的加载器
（1）根类加载器（BootStrap 启动类加载器），没有父类加载器,由C++实现，不是ClassLoader的子类
（2）扩展类加载器（Extension），父类加载器是根类加载器
（3）系统（应用）类加载器（System），父类加载器是扩展类加载器

注：除根类加载器没有父类加载器外，其他加载器都有且仅有一个父类加载器。当Java程序请求加载器时，首先会去请求其父类加载器，若父类加载器能完成加载，则由父类加载器完成，若不能，再由子类加载器完成，这种模式称为双亲加载机制。

2）用户自定义的类加载器
（1）java.lang.ClassLoader的子类
（2）用户可以定制类的加载方式

注：所有的用户自定义加载器都继承自CLassLoad类
注：类加载器并不需要等到某个类被“首次主动使用”时再加载它，

4、jvm规范允许类加载器再预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）

5、如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误。

### 类的验证
1、类被加载后，进入连接阶段。连接就是将已经读入到内存中的类的二进制数据合并到虚拟机的运行时环境中去。

2、类的验证的主要内容
（1）类文件的结构检查
（2）语义检查
（3）字节码验证
（4）二进制兼容性的验证

### 类的初始化
步骤：
1）假如类还没有加载和连接，那么就先加载和连接
2）假如类存在父类，并且父类还没有初始化，那就先初始化直接父类（不适用于接口）
3）假如类中存在初始化语句，那就依次执行这些初始化并语句  

### 类加载器的双亲委托机制
1、在双亲委托机制中，各个加载器按照父子关系形成了逻辑上的树形结构（物理上没有关系），除了根类加载器之外，其余的类加载器都有且仅有一个父加载器
2、当Java程序请求加载器时，首先会去请求其父类加载器，若父类加载器能完成加载，则由父类加载器完成，若不能，再由子类加载器完成，这种模式称为双亲委托机制。（若父类再有父类，就继续委托给父类，层层往上委托一直到根类加载器）
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/jvmclassload1.png?raw=true)
3、定义类加载器：能够成功加载你所要加载类的加载器
4、初始化加载器：所有能成功返回Class对象引用的类加载器（包括定义类加载器）

例子：
```java
package jvm;

public class ClassLoadTest {

	public static void main(String[] args) throws ClassNotFoundException {
		//得到java.lang.String对象
		Class<?> clazz=Class.forName("java.lang.String");
		//得到String对象的类加载器
		System.out.println(clazz.getClassLoader());
	}
}
```
输出:
```java
null
```
若一个类的类加载器是根加载器，则用null来表示，因为根加载器是由C++实现的（不过不同版本jvm有不同的表示）。
由输出知：String类的加载器是根加载器

例子：
```java
package jvm;

public class ClassLoadTest {

	public static void main(String[] args) throws ClassNotFoundException {
		//得到LittleTest对象
		Class<?> clazz=Class.forName("jvm.LittleTest");
		//得到LittleTest对象的类加载器
		System.out.println(clazz.getClassLoader());
	}
}
//自定义内部类LittleTest
class LittleTest{
	
}
```
输出:
```java
sun.misc.Launcher$AppClassLoader@2a139a55
//AppClassLoader应用类加载器
```
由输出知：内部类LittleTest的类加载器为应用（系统）类加载器