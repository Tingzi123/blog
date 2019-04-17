title: JVM内存模型
---
## JVM内存模型
jvm启动流程：
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/jvm00.png?raw=true)

### 1、JVM = 类加载器(classloader) + 执行引擎(execution engine) + 运行时数据区域(runtime data area)
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/jvmmey1.png?raw=true)

### 2、运行时数据区域　
Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则是依赖用户线程的启动和结束而建立和销毁。
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/jvmmey2.png?raw=true)

### 3、程序计数器（Program Counter Register）
线程私有，它的生命周期与线程相同。
可以看做是当前线程所执行的字节码的行号指示器。
在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，如：分支、循环、跳转、异常处理、线程恢复（多线程切换）等基础功能。
如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie方法，这个计数器值则为空（undefined）。
程序计数器中存储的数据所占空间的大小不会随程序的执行而发生改变，所以此区域不会出现OutOfMemoryError的情况。

### 4、Java虚拟机栈（JVM Stacks）
线程私有的，它的生命周期与线程相同。
虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

1）局部变量表
包含参数和局部变量，存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型），它不等同于对象本身，根据不同的虚拟机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。
该区域可能抛出以下异常：
当线程请求的栈深度超过最大值，会抛出 StackOverflowError 异常；
栈进行动态扩展时如果无法申请到足够内存，会抛出 OutOfMemoryError 异常。

2） 操作数栈
Java没有寄存器，所有的参数传递使用操作数栈

3）Java栈上分配
如果是小对象（一般几十个bytes），在没有逃逸的情况下，可直接分配在栈上，减轻堆的压力

### 5、本地方法栈（Native Method Stacks）
与虚拟机栈非常相似，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则是为虚拟机使用到的Native 方法服务。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun HotSpot 虚拟机）直接就把本地方法栈和虚拟机栈合二为一。
与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

### 6、Java堆（Heap）
被所有线程共享，在虚拟机启动时创建，用来存放对象实例，几乎所有的对象实例都在这里分配内存。
对于大多数应用来说，Java堆（Java Heap）是Java虚拟机所管理的内存中最大的一块。
Java堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC堆”。如果从内存回收的角度看，由于现在收集器基本都是采用的分代收集算法，所以Java堆中还可以细分为：新生代和老年代；新生代又有Eden空间、From Survivor空间、To Survivor空间三部分。
Java 堆不需要连续内存，并且可以通过动态增加其内存，增加失败会抛出 OutOfMemoryError 异常。
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/jvmmey3.png?raw=true)

### 7、方法区（Method Area）
用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
和 Java 堆一样不需要连续的内存，并且可以动态扩展，动态扩展失败一样会抛出 OutOfMemoryError 异常。
对这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，但是一般比较难实现，HotSpot 虚拟机把它当成永久代（Permanent Generation）来进行垃圾回收。
方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。
> 运行时常量池（Runtime Constant Pool）
---
运行时常量池是方法区的一部分。
Class 文件中的常量池（编译器生成的各种字面量和符号引用）会在类加载后被放入这个区域。
除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()。这部分常量也会被放入运行时常量池。
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/jvmmey4.png?raw=true)
注：
在 JDK1.7之前，HotSpot 使用永久代实现方法区；HotSpot 使用 GC 分代实现方法区带来了很大便利；
从 JDK1.7 开始HotSpot 开始移除永久代。其中符号引用（Symbols）被移动到 Native Heap中，字符串常量和类引用被移动到 Java Heap中。
在 JDK1.8 中，永久代已完全被元空间(Meatspace)所取代。元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。
直接内存（Direct Memory）
直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError 异常出现。
在 JDK 1.4 中新加入了 NIO 类，引入了一种基于通道（Channel）与缓冲区（Buffer）的 I/O方式，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆里的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java 堆和 Native 堆中来回复制数据。

栈、堆、方法区交互
![Image text](https://github.com/Tingzi123/blog/blob/master/_posts/picture/jvm01.png?raw=true)

### 可见性
一个线程修改了变量，其他线程可以立即知道

保证可见性的方法：
1）volatile
2）synchronized（unlock之前，写变量值回主存）
3）final（一旦初始化完成，其他线程就可见）

参考：http://baijiahao.baidu.com/s?id=1598140630731512683&wfr=spider&for=pc