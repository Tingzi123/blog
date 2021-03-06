title: Thinking in Java - 对象导论
---
1、抽象过程
1）万物皆为对象
2）程序是对象的集合，他们通过发送消息来告知彼此所要做的
要想请求一个对象，就必须对该对象发送一条消息，可以把消息想象为对某个特定对象的方法的调用请求
3）每个对象都有自己的由其他对象所构成的存储
4）每个对象都拥有其类型
5）某一特定类型的所有对象都可以接受同样的消息

对象具有状态、行为和标识。这意味着每一个对象都可以拥有内部数据（状态）和方法（行为），都可以唯一的与其他对象区分开来，即每一个对象在内存中都有一个唯一的地址。

2、每个对象都有一个接口
1）关键字class的由来:在程序执行期间具有不同状态而其它方面都相似的对象会被分组到对象的类中
2）一个类实际上就是一个数据类型，因为类描述具有相同特性（数据元素)和行为（功能）的对象集合
3）class，是Java编程语言中的基本单位

3、每个对象都提供服务
1）将对象想像为“服务提供者”，有助于提供对象的内聚性

4、被隐藏的具体实现（访问权限）
1）public：任何人可用
2）private：除类创建者和类的内部方法之外的任何人都不能访问
3）protected：类创建者和类的内部方法能访问，继承的类也能访问protected成员，不能访问private成员
4）包访问权限：默认的访问权限，类可以访问同一个包（库构件）中的其他类成员，对于包之外，该包内的成员相当于被private所修饰

5、复用的具体实现
1）最简单的复用某个类：直接使用该类的对象
2）组合：将该类的对象置于某个新的类中，新的类可以由任意数量、任意类型的其他对象以任意可以实现新的类中想要的功能的方式所组成。

6、继承（是is-a的关系)
1）当父类发生变动时，子类也会反映出这种变动
2）类不仅仅只是描述作用于一个对象集合上的约束条件，还有与其他类之间的关系。两个类可以有相同的特性和行为，但其中一个类可能比另一个含有更多的特性，并且可以处理更多的消息，继承使用父类和子类的概念表示了这种类之间的相似性。
3）一个父类包含其所有子类所共享的特性和行为。
4）继承现有类时，也就创造了新的类。新的类包括现有类的所有成员（包括不可访问的private成员），并且复制了父类的接口，即所有可以发送给父类对象的消息同时也可以发送个子类的对象。
5）由于通过发送给类的消息的类型可知类的类型，则这意味着父类与基类具有相同的类型

6.1 使父类与子类产生差异的方法
1）直接在子类中添加新方法，父类不能访问这个新方法，是一种is-like-a的关系
2）覆盖

7、伴随多态的可互换对象
1）处理类型的层次结构时，经常想把一个对象不当作它所属的特定类型来看待，而是当作其父类来看待。
2）这使得可以编写出不依赖于特定类型的代码，这样的代码不会受新添加的类型的影响
3）前期绑定：非面向对象编程的编译器产生的函数调用会引起前期绑定，编译器将产生对一个具体函数名字的调用，运行时将这个调用解析到将要被执行的代码的绝对地址。
4）后期绑定：在OOP中，程序直到运行时才能确定代码的地址。当向对象发送消息时，被调用的代码直到运行时才能确定。编译器确保被调用方法的存在，并对调用参数和返回值执行类型检查，但是不知道将被执行的确切代码。
5）为了后期绑定，Java用一小段特殊代码来替代绝对地址调用，这段代码使用在对象中存储的信息来计算方法体的地址。根据这一小段代码的内容，每一个对象都可以具有不同的行为表现。
5）在Java中，动态绑定是默认行为，不需要添加额外的关键字来实现多态。

8、单根继承结构
1）Java中所有的类最终都继承自单一的类：Object
2）单根继承使垃圾回收器的实现变得容易很多。
3）所有的对象都可以很容易在堆上创建，参数传递也得到极大简化
4）所有的对象都保证具有其类型信息，因而不会因无法确定对象的类型而陷入僵局，这对于系统级操作（如异常处理）尤其重要，并且给编程带来更大的灵活性。

9、容器
1）