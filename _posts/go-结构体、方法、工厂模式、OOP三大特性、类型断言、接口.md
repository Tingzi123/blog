---
title: 'go:结构体、方法、工厂模式、OOP三大特性、类型断言、接口'
date: 2020-06-11 15:35:04
tags:
---
### 面向对象-结构体
1、GO支持面向对象编程（OOP），但不是纯粹的面向对象编程语言，更准确地说是**GO支持面向对象编程特性**
2、GO没有类的概念，结构体（struct）来实现OOP
3、GO面向对象编程非常简洁，去掉传统OOP语言的继承、方法重载、构造函数和析构函数、隐藏的this指针等
4、Go仍然有面向对象编程的继承、封装和多态的特性，只是实现方式不一样。
5、GO面向对象（OOP）很优雅，OOP本身就是语言类型系统的一部分（Go天然支持OOP），通过接口（interface）关联，耦合性低，非常灵活。
6、GO更准确的说是**面向接口编程**

##### 1、结构体变量在内存中的布局
```go
//定义一个结构体
type Cat struct{
    Name string
    Age int
    Color string
    Hobby string
}

//创建一个结构体cat变量
vat cat1 Cat
cat.Name = "小白"
cat.Age = 3
cat.Color = "白色"
cat.Hobby = "吃鱼"

fmt.Println("cat1=",cat1)
```
结构体是自定义的数据类型(如Cat)，结构体变量（实例）是具体的、实际的一个变量(如cat1)

结构体实例在内存中的布局，以cat1为例
1、当代码执行到“var cat1 Cat”时，cat1指向一个结构体，这时还没有给结构体的各个字段赋值，所以各个字段为默认值
![struct1](./struct1.jpg)

2、结构体有一个自己的地址，cat1直接指向一个结构体的数据空间，而不是结构体的地址，所以**结构体是值类型**

3、赋值语句为结构体的字段赋值
![struct2](./struct2.jpg)

总结：

* 当我们声明一个结构体变量时，结构体的数据空间已经有了，且结构体的每个字段已有**默认值**
* 声明结构体变量后，该变量指向一个结构体的数据空间，而不是该结构体数据空间的地址，所以结构体是**值类型**

##### 2、结构体声明
1、在创建一个变量后，如果没有给字段赋值，系统会赋零值

* 基本数据类型赋零值
* **引用类型：slice、map、指针的零值是nil，即没有分配内存空间**
* 数组类型的默认值与元素相关

2、不同结构体变量的字段你是独立的，互不影响。因为结构体是值类型、默认是值拷贝

##### 3、创建结构体实例的4种方式
1、直接声明
```go
    vat person Person
    person.field1 = value1
    person.field2 = value2
    person.field3 = value3
```
2、声明时直接为字段赋值
```go
    vat person Person = Person{
        field1:value1
        field2:value2
        field3:value3
    }
```

3、通过new函数创建，创建对象是一个指向该结构体的指针
```go
    vat person *Person = new(Person)
    //person是一个指针，标准赋值方式如下
    (*person).field1 = value1  
    (*person).field2 = value2
    (*person).field3 = value3
    //运算符优先级："." > "*"，所以需要括号(*person).field1
    
    //可简写为
    person.field1 = value1
    person.field2 = value2
    person.field3 = value3
    
    //GO的设计者为了使用方便，底层会对简写写法person.field1 = value1进行处理，会给person加上取值运算 (*person).field1 = value1
```

4、vat person *Person = &Person{}，可以直接再{}中赋值，如方式2，也可如下赋值，如方式3
```go
    vat person *Person = &Person{}

//person是一个指针，标准赋值方式如下
    (*person).field1 = value1
    (*person).field2 = value2
    (*person).field3 = value3
    
    //可简写为
    person.field1 = value1
    person.field2 = value2
    person.field3 = value3
    
    //GO的设计者为了使用方便，底层会对简写写法person.field1 = value1进行处理，会给person加上取值运算 (*person).field1 = value1
```

总结:

* 方式3和方式4返回的是一个**结构体指针**

##### 4、结构体内细节
1、**结构体中所有字段在内存中是连续分布的**
2、当结构体的字段是指针时，指针本身的地址是连续的，但是指针指向的地址指值不一定连续
3、结构体使用户单独定义的类型，和其它类型转换时，需要有完全相同的字段（名字、个数和累型）
4、结构体进行type重新定义（相当于取别名），Go认为是**新的数据类型**，但是相互间可以强转
5、在结构体的每个字段上，可以写上一个**tag**，该tag可以通过**反射机制**获取，常见的使用场景就是**序列化**和**反序列化**
```go
    type Monster struct{
        Name string `json:"name"`
        Age int `json:"age"`
        Skill string `json:"skill"`
    }

    monster := Monster{
        "牛魔王"，
        500，
        “牛头拳”
    }
    
    //将monster变量序列化为json字符串（反射机制）
    jsonStr,err := json.Marshal(monster)//返回值为[]byte切片
    //这里json包去访问monster中的字段，如果字段小写则访问不到，字段小写表示只能当前包内访问，则只能大写。但是大写，json格式化后json串中也是大写，与前端命名习惯不一致，所以可以用tag为字段取别名成小写形式
    
    if err != nil{
        fmt.Println("json处理错误"，err)
    }
    fmt.Println("jsonStr",string(jsonStr))
```

### 方法method
1、Go中方法是**作用在指定的数据类型上**的（和指定数据类型绑定），因此自定义类型，都可以有方法（不仅是struct）。

2、方法的声明与调用
```go
    type A struct{
        Num int
    }

    func (a A) test(){//A结构体有一个方法test，说明test方法和结构体（类型）A绑定，test方法只能被A的对象来调用
    
        a.Num=2 //这里的改变不会影响main函数中的赋值输出，因为struct是值类型，func (a A)这里不是指针。
        fmt.Println(a.Num)
    }
    
    func main(){
        var a A 
        a.Num=1
        a.test()//调用方法
    }
```
   由上述代码有：

*    1、test方法和A类型绑定
*    2、test方法只能通过A的实例来调用，**不能直接调用**，也不能用其他类型调用
*    **3**、func (a A) test()，**a表示哪个A实例调用，他就是该实例的副本**，和函数传参类似，由于**struct是值类型**，func (a A)这里不是指针，所以方法中对字段重新赋值不会影响main函数中的值

#### 方法的调用和传参机制
**重点**：方法的调用和传参机制和函数基本一样，不一样的地方是方法调用时，**会将调用方法的实例变量，当作实参也传递给方法**

示例：
```go
    type Person struct{
        Name string
    }

    func (p Person) getSum(n1 int,n2 int) int{
        return n1+n2
    }
    
    func main(){
        var p Person 
        p.Name = "tom"
        
        n1 := 10
        n2 := 20
        res := p.getSum(n1,n2)//调用方法
        
        fmt.Println("res=",res)
    }
```
1、main函数中执行“n1 := 10 n2 := 20”时，内存中开辟main栈并压入变量n1和n2
![method1](./method1.png)

2、main函数执行 “ res := p.getSum(n1,n2)”时：

* 先执行“p.getSum(n1,n2)”，内存中开辟getSum栈，并将n1和n2的值拷贝到getSum栈中，同时将结构体实例p值拷贝到getSum栈中（因为结构体实例p调用方法getSum），因为这里传的是“func (p Person)”，所以是值拷贝，所以main栈和getSum栈是完全独立的数据空间。若传指针则为引用传递
![method2](./method2.png)


* 然后执行赋值给res变量“res := p.getSum(n1,n2)”，main栈中压入res变量，res变量的值有函数getSum返回
![method3](./method3.png)

注意：**如果一个类型（结构体）实现了String()这个方法，fmt.Println默认会调用这个结构体变量的String()方法输出**

#### 方法和函数的区别
1、调用方式不一样

* 函数的调用方式：函数名（实参列表）
* 方法的调用方式：绑定类型变量.方法名（实参列表）

2、对于普通函数，接收者为值类型时，不能将指针类型数据直接传递，反之亦然

3、对于方法（如struct的方法），接收者为值类型时，可以直接用指针类型的变量调用方法，反之亦然
 ```go
 type Person struct{
        Name string
    }

    func (p Person) test() {
        p.Name = "jack"
        fmt.Println("tes=t",p.Name) //jack
    }
    
     func (p *Person) test01() {
        p.Name = "marry"
        fmt.Println("tes=t",p.Name) //marry
    }
    
    func main(){
        var p Person 
        p.Name = "tom"
        
        p.test() //ok
        fmt.Println("tes=t",p.Name) //tom
        
        (&p).test()  //用p的地址（指针）调也可以，但仍然是值拷贝，只是形式上看是引用拷贝，但是接受的地方“func (p Person)”是传值，所以底层会处理为“p.test()”，只有接受的地方为传地址（如 func (p *Person)），才是引用传递
        fmt.Println("tes=t",p.Name) //tom
        
        (&p).test01()//ok
        fmt.Println("tes=t",p.Name)//marry
        
        p.test01()//ok，等价于(&p).test01()，底层处理，形式上传值，实际传地址，因为接受的地方为传地址（如 func (p *Person)）
        fmt.Println("tes=t",p.Name)//marry 
    }
```

**针对第3条总结**：**重点重点**
不管调用的形式如何，真正决定是值拷贝还是地址拷贝，看定义这个方法时跟哪个类型绑定，若跟指针（如 p *Person）绑定则为地址拷贝，若跟值类型（如 p Person）绑定则为值拷贝

#### 工厂模式
1、GO的结构体没有**构造函数**，通常使用工厂模式来解决

* 使用工厂模式实现跨包创建结构体实例

1）model包下有结构体Student
 ```go
    package model 

     type student struct{
     //这里结构体定义为小写student，只能在当前包内使用，其他包不可访问
            Name string
            Score float64
       }
        
      //使用工厂模式
      func NewStudent(name string,score float64) *student{
          return &student{
              Name : name,
              Score : score,
          }
      }
```
2）main包下使用工厂模式创建结构体student实例
 ```go
    package main
    import (
        "fmt"
        "factory/model"
       )
       
    func main(){
        //使用工厂模式创建student实例
        var stu = model.NewStudent("tom",88.8)//返回值为一个指针
       
       fmt.Println(stu) //输出&{tom 88.8}
       fmt.Println(*stu) //输出{tom 88.8}
       
       fmt.Println(stu.Name) //等价于(*stu).Name 输出tom
       fmt.Println(stu.Score) //等价于(*stu).Score 输出88.8
    }
```

* 如果结构体中的字段也是小写，只能在定义结构体的包中使用，其他包不可访问，同理可以为结构体绑定一个公有方法，用于返回字段
1）model包下有结构体Student
 ```go
    package model 

     type student struct{
     //这里结构体定义为小写student，只能在当前包内使用，其他包不可访问
            Name string
            score float64
     //这里字段score定义为小写，只能在当前包内使用，其他包不可访问
       }
        
      //使用工厂模式
      func NewStudent(name string,score float64) *student{
          return &student{
              Name : name,
              score : score,//该包内可以访问小写字段，其他包不可以
          }
      }
      
      //GetScore方法绑定结构体，返回字段score
      func (s *student) GetScore() float64{
            return s.score
      }
```
2）main包下使用工厂模式创建结构体student实例
 ```go
    package main
    import (
        "fmt"
        "factory/model"
       )
       
    func main(){
        //使用工厂模式创建student实例
        var stu = model.NewStudent("tom",88.8)//返回值为一个指针
       
       fmt.Println(stu) //输出&{tom 88.8}
       fmt.Println(*stu) //输出{tom 88.8}
       
       fmt.Println(stu.Name) //等价于(*stu).Name 输出tom
       fmt.Println(stu.GetScore()) //输出88.8
    }
```

### 面向对象编程三大特性

面向对象编程三大特性：**封装、继承、多态**。

#### 1、封装
1、封装：把**抽象出来的字段和对字段的操作封装在一起**，数据就被保护在内部，程序的其他包只有通过被授权的操作（方法），才能对字段进行操作

2、封装的优点

* 隐藏实现的细节
* 提供方法可以对数据进行验证、保证安全合理

3、如何体现封装

* 对结构体中的属性进行封装
* 通过方法、包实现封装

4、封装的实现
参考上一个笔记，工厂模式（将结构体、属性定义为包私有，通过公有方法访问）

#### 2、继承
继承可以**解决代码复用**，提高扩展性、可维护性
![oop1](./oop1.jpg)
GO使用在一个结构体中**嵌套匿名结构体**的方式来实现继承
案例：
1、抽取共有字段和方法，创建一个结构体Student
```go
     type Student struct{
           Name string
           Age int
           Score float64     
       }
       
       //共有方法
       func (stu *Student) ShowInfo(){
            fmt.Printf("学生名=%v年龄=%v 成绩=%v           \n",stu.Name,stu.Age,stu.Score)

        }
        
        func (stu *Student) SetScore(score int){
            stu.Score = score
        }
```

2、创建结构体Pupil
```go
     type Pupil struct{
           Student    //嵌入Student匿名结构体
       }
       
       //Pupil特有方法
       func (p *Pupil) testing(){
            fmt.Printf("小学生正在考试中")

        }
```

3、创建结构体Graduate
```go
     type Graduate struct{
           Student    //嵌入Student匿名结构体
       }
       
       //Graduate特有方法
       func (g *Graduate) testing(){
            fmt.Printf("大学生正在考试中")

        }
```

4、main创建实例
```go
       func mian(){
            //当我们对结构体嵌入了匿名结构体后，使用方法发生了变化
            //创建一个Pupil实例
            pupil := &Pupil{}
            pupil.Student.Name = "tom"
            pupil.Student.Age = 8
            pupil.testing()
            pupil.Student.SetScore(70)
            pupil.Student.ShowInfo()
            
            //创建一个Graduate实例
            graduate := &Graduate{}
            graduate.Student.Name = "marry"
            graduate.Student.Age = 8
            graduate.testing()
            graduate.Student.SetScore(70)
            graduate.Student.ShowInfo()
        }
```
##### 继承细节
1、结构体可以使用嵌套匿名结构体的**所有字段和方法**，即：首字母大写和小写的字段和方法都可以使用

2、访问匿名结构体字段可以简化
在上述案例中
```go
       func mian(){
            //当我们对结构体嵌入了匿名结构体后，使用方法发生了变化
            //创建一个Pupil实例
            pupil := &Pupil{}
            pupil.Student.Name = "tom"
            pupil.Student.Age = 8
            pupil.testing()
            pupil.Student.SetScore(70)
            pupil.Student.ShowInfo()
            
            //创建Pupil实例，访问Pupil结构体中的匿名结构体Student的字段和方法可以简化为
            pupil := &Pupil{}
            pupil.Name = "tom"
            pupi.Age = 8
            pupil.testing()
            pupil.SetScore(70)
            pupil.ShowInfo()
            
            //pupil.Student.Name = "tom" 等价于pupil.Name = "tom"，
           // pupil.Name 先去pupil找Name字段，如果没有，再去Pupil结构体中的匿名结构体Student找字段Name
           //如果找到则访问，否则报错
        }
```

3、当结构体和匿名结构体有相同的字段或方法时，编译器采用**就近访问原则**访问，如希望访问匿名结构体的字段和方法，可以通过匿名结构体来区分
```go
type A struct{
    Name string
    age int
}

func (a *A) SayOk(){
    fmt.Println("A SayOK",a.Name)
}

func (a *A) hello(){
    fmt.Println("A hello",a.Name)
}
```

```go
type B struct{
    A
    Name string
}

func (b *B) SayOk(){
    fmt.Println("b SayOK",b.Name)
}
```

```go
       func mian(){
          var b B
          
          b.Name = "jack"//ok
          //去结构体B中找字段Name，找到并赋值jack，则b.Name= "jack"
          
          b.SayOK() //b SayOK jack
          //去结构体B中找方法SayOK ，找到并执行fmt.Println("b SayOK",b.Name)，这里b.Name= "jack"
          
          b.hello() //A hello ""空串
          //去结构体B中找方法hello，没有找到，再去匿名结构体A中找，找到并执行fmt.Println("A hello"，a.Name)，前面的语句并没有为a.Name赋值，则为默认值空串，a.Name就近等于空串
          
          b.A.Name = "tom"//ok
          //去结构体B的匿名结构体A中找字段Name，找到并赋值tom，则b.Name= "tom"
          
          b.SayOK() //b SayOK jack
          //去结构体B中找方法SayOK ，找到并执行fmt.Println("b SayOK",b.Name)，这里b.Name= "jack"
          
          b.A.SayOK() //A SayOK tom
          //去结构体B的匿名结构体A中找方法SayOK ，找到并执行fmt.Println("A hello",a.Name)，这里a.Name= "tom"
          
          b.hello() //A hello tom
          //去结构体B中找方法hello，没有找到，再去匿名结构体A中找，找到并执行fmt.Println("A hello"，a.Name)，前面的语句为a.Name赋值tom，则a.Name就近等于tom
       
      }
```
4、当结构体嵌入两个（或多个）匿名结构体，如**两个匿名结构体有相同的字段和方法（同时结构体本身没有同名的字段和方法）**，在访问时，就必须要明确指定匿名结构体的名字，否则编译报错

5、如果一个struct嵌套了一个有名字的结构体，这种模式就是**组合**，如果是组合关系，那么在访问组合的结构体的字段和方法时，就必须带上结构体的名字，**不能简写**。
Go中层级关系明确，如果是组合关系，例：一个结构体A嵌套了一个有名结构体B，B的名字为b,B有字段Name，现创建A的实例a,访问字段Name时，只能a.b.Name，不能a.Name，编译器会报错。看到a.Name时，编译器会去A中找字段Name，没有就立马报错（嵌套匿名结构体时会继续在匿名结构体中找）

6、嵌套匿名结构体后，也可以创建结构体变量时，直接指定各个匿名结构体字段的值
```go
pupil := &Pupil{Student{"tom"，15，78.5}}

//或者
pupil := &Pupil{
    Student{
        Name:"tom"，
        Age:15，
        Score:78.5，
     }
 }
```

7、在结构体中嵌入匿名结构体时，也可以直接嵌入匿名结构体的指针，这样效率更高

8、在结构体中嵌入匿名结构体时，也可以直接嵌入匿名基本数据类型。
如嵌入匿名int，访问时直接通过 “外层结构体实例名.int = 20”这种形式。但是不能同时侵入两个同样的匿名基本数据类型，因为无法区分，如嵌入两个匿名int，这是不允许的。

##### 多重继承
一个struct嵌套了多个匿名结构体，那么该结构体可以直接访问所有嵌套的匿名结构体的字段和方法，从而实现多重继承
1、当结构体嵌入两个（或多个）匿名结构体，如**两个匿名结构体有相同的字段和方法（同时结构体本身没有同名的字段和方法）**，在访问时，就必须要明确指定匿名结构体的名字，否则编译报错

2、为了代码简洁性，建议尽量不适用多重继承

#### 3、多态
##### 1、基本介绍
1、变量（实例）具有多种形态，在GO中，多态特征是通过接口实现的。可以按照统一的接口来调用不同的实现，这时**接口变量就呈现不同的形态**。
案例说明：
1、定义一个接口Usb
```go
    type Usb interface{
        //声明两个没有实现的方法
        Start()
        Stop()
    }
```
2、定义一个结构体Phone
```go
      type Stu struct{
    }
    
    //让Phone实现接口Usb的所有方法，即实现了接口Usb
    func (p Phone) Start(){
        fmt.Println("手机开始工作。。。")
    }
    
    func (p Phone) Stop(){
        fmt.Println("手机停止工作。。。")
    }
```
3、再定义一个结构体Camera
```go
    type Stu Camera{
    }
    
   //让Camera实现接口Usb的所有方法，即实现了接口Usb
    func (c Camera) Start(){
        fmt.Println("相机开始工作。。。")
    }
    
    func (c Camera) Stop(){
        fmt.Println("相机停止工作。。。")
    }
```
4、再定义一个结构体Computer
```go
    type Stu Computer{
    }
    
    //编写一个方法Working，接收一个Usb接口类型变量
    func (c Computer) Working(usb Usb){
        //通过usb接口变量来调用Start和Stop方法
        usb.Start()
        usb.Stop()
    }
```

5、在main函数中使用接口
```go
    func main(){
        //创建结构体Computer变量
        computer := Computer{}
        
         //创建结构体Phone变量
        phone := Phone{}
        
        //创建结构体Camera变量
        camera := Camera{}
        
        //关键点
        computer.Working(phone)
        //手机开始工作。。。
        //手机停止工作。。。
        
        computer.Working(camera)
         //相机开始工作。。。
        //相机停止工作。。。
        
        //结构体Computer的Working函数接收的是一个Usb接口变量，因为结构体Phone和Camera都实现了这个接口，所以可以传入，并且当传Phone时，可以动态的调用phone.Start()和phone.Stop()，传入camera时同理。Usb接口变量体现出多态。
    }
```
##### 2、接口体现多态的特征
1、多态参数
在前面的Usb接口案例中，Computer的Working函数接收的是一个Usb接口变量，Phone和Camera都实现了这个接口，所以既可以接收Phone变量，又可以接收Camera变量，体现出多态。

2、多态数组
在上述Usb案例中，我们可以定义一个Usb接口数组
```go
    func main(){
        //定义一个Usb接口数组，可以存放Phone和Camera的结构体变量
        var usbArr [3]Usb
        fmt.Println(usbArr)//输出[<nil><nil><nil>]
        
        usbArr[0] = Phone{}
        usbArr[1] = Phone{}
        usbArr[2] = Camera{}
        fmt.Println(usbArr)//输出[<><><>]
        
        //上述的案例中的结构体都没有字段，这里都加上Name字段，然后创建结构体变量
        usbArr[0] = Phone{"华为"}
        usbArr[1] = Phone{"小米"}
        usbArr[2] = Camera{"索尼"}
        fmt.Println(usbArr)//输出[<华为><小米><索尼>]
        
        //Usb接口数组usbArr，可以存放Phone和Camera的结构体变量，体现出多态数组
    }
```

### 类型断言
##### 1、基本介绍
1、**类型断言**，由于**接口**是一般类型，**不知道具体类型**。如果要转成具体类型，就需要使用类型断言。

##### 2、案例
1、定义结构体Point
```go
type Point struct{
    x int
    y int
}
```

2、main函数
```go
    func main(){
        var a interface{} //定义一个空接口类型变量a
        var point Point = Point{1,2} //定义Point类型变量point
        
        a = point //ok，
        //将Point类型变量point赋给一个空接口类型变量a，
        //使空接口类型变量a指向Point类型的变量point
        //空接口变量可以接收任何数据类型
        
        var b Point
        //b = a不可以，不可以将一个空接口类型变量a赋给其他数据类型
        //b = a.(Point)可以，a.(Point)称为类型断言
        
        b,ok = a.(Point) 
        //判断空接口类型变量a是指向Point类型的变量
        //如果是就转成Point类型的变量并赋给b，否则报错
        
        //类型断言检查，不要让断言失败直接panic终止程序
        if ok {
            fmt.Println(b)//输出<1，2>
        }else{
             fmt.Println("转换失败")
         }       
         
         fmt.Println("代码继续执行")//类型断言检查，断言失败不会panic终止程序，这里还会执行到
    }
```
**重点**：

* 在进行类型断言时，如果类型不匹配，就会报panic。因此进行类型断言时，要确保原来的空接口指向的就是要断言的类型
* 例：首先“a = point ”使空接口类型变量a指向Point类型的变量point，才可以类型断言“b = a.(Point) ”

##### 3、类型断言最佳案例
![assert1](./assert1.png)


### 接口
##### 1、基本介绍
1、interface类型可以定义一组方法但是不需要实现

2、interface中**不能包含任何变量**

##### 2、基本语法
```go
type 接口名 interface{
    method1(参数列表)返回值列表
    method2(参数列表)返回值列表
    ...
}

type 自定义类型名 struct{
    
}

func (s 自定义类型名) method1(参数列表)返回值列表{
    //方法实现
}

func (s 自定义类型名) method2(参数列表)返回值列表{
    //方法实现
}

...
```
注意：
1）接口中的所有方法都没有方法体，即接口的方法都是没有实现的方法。体现**多态**、**高内聚低耦合**

2）Go中的接口，不需要显示实现。只要一个结构体变量（实例）含有接口类型中的所有方法，则称这个结构体变量实现了这个接口（没有类似implement这样的关键字，GO是基于方法实现的多态，而不需显示指出接口名称）。

3）如果由接口a和接口b有完全一样的方法列表，那么有实例c如果实现了接口a，同时也就实现了接口b，即：同时实现两个或两个以上的接口（在其他语言中是做不到的，例如Java，必须显示implement实现a，再显示implement实现b）

4）特别指出，需要**实现接口所有的方法**

##### 3、注意事项
1、**接口本身不能创建实例**，但是可以指向一个实现了该接口的自定义类型（如struct）的变量（实例）

2、接口中的所有方法都没有方法体，即接口的方法都是没有实现的方法

3、Go中，一个自定义类型需要将某个接口的**所有方法都实现**，才能称这个自定义类型实现了该接口

4、只要是自定义类型，都可以实现接口，不仅仅是结构体类型。如自定义一个int类型“type integer int”，这个自定义的类型integer也可以实现接口

5、一个自定义类型可以**实现多个**接口。（只要把多个接口的方法都实现就可以）

6、Go接口中不能有任何变量（常量也不可以）

7、一个接口（如接口A）可以**继承多个**别的接口（如接口B和C），这时如果要实现接口A，也必须要将接口B和C的方法全部实现，但是**接口B和C中不能有完全相同的方法**，编译器会报错重复定义，因为无法区分。

8、interface类型默认是一个**指针**（**引用类型**），如果没有对interface初始化就使用，那么会输出nil

9、空接口interface{}，没有任何方法，所以**所有类型都默认实现了空接口**（即空接口可以接收任何数据类型，可以把任何数据类型变量赋给空接口）。

10、**空接口也是一种数据类型**
例：可以声明一个空接口类型变量
“vat t interface{}”

该空接口类型变量可以接收任何数据类型的值
“t = student”//接收一个Student类型的变量student
“t = 8.8”//接收一个float类型的值8.8

##### 4、接口使用案例
1、定义一个接口AInteger
```go
    type AInteger interface{
        Test01()
        Test02()
    }
```
2、再定义一个接口BInteger
```go
    type AInteger interface{
        Test01()
        Test03()
    }
```
3、定义一个结构体Stu
```go
    type Stu struct{
    }
    
    //实现方法Test01
    func (stu Stu) Test01(){
    }
    
    //实现方法Test02
     func (stu Stu) Test02(){
    }
    
    //实现方法Test03
      func (stu Stu) Test03(){
    }
```
4、在main函数中使用接口
```go
    func main(){
        stu := Stu{}//创建一个Stu实例
        
        var a AInteger =stu 
        //ok，将实例stu赋给接口AInteger的变量a，因为stu实现了接口AInteger中的所有方法（ Test01和Test02）
        
        var b BInteger =stu
        //ok，将实例stu赋给接口BInteger的变量b，因为stu实现了接口BInteger中的所有方法（ Test01和Test03）
        
        fmt.Println("ok",a,b)//ok
    }
```
##### 5、接口与继承比较
1、当B结构体继承了A结构体，那么B结构体就自动的继承了A结构体的字段和方法，并且可以直接使用

2、B结构体需要扩展功能，同时不希望破坏继承关系，可以去实现某个接口，因此，可以认为：实现**接口是对继承机制的一种补充**

3、接口与继承解决的问题不同

* 继承的价值：解决代码的**复用性**和**可维护性**
* 接口的价值：**设计**，设计好各种规范（方法），让其他自定义类型去实现这些方法来增强功能

4、接口比继承更加灵活。**继承**是满足 **is - a** 的关系，**接口**只需要满足 **like - a** 的关系

5、接口在一定程度上实现代码解耦，尤其在GO中更加松散