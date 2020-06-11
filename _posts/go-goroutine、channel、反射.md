---
title: 'go:goroutine、channel、反射'
date: 2020-06-11 15:58:15
tags:
---
### 1、goroutine
#### 1、基本介绍
1、Go协程和Go主线程

* **Go主线程**（有的程序员直接成为线程/也可以理解成进程）：**一个Go主线程上，可以起多个协程**，即**协程是轻量级的线程**。

2、Go协程的特点

* 有独立的栈空间
* 共享程序堆空间
* 调度由用户控制
* 协程是轻量级的线程

3、案例
1）在主线程（可以理解成进程），开启一个goroutine，该协程每隔一秒输出“hello world”
2）在主线程中也每隔一秒输出“hello world”，输出10次后，退出程序
3）要求主线程和goroutine同时执行
4）主线程和协程执行流程图

```go
package main
import (    
    "fmt"    
    "strconv"   
    "time"
)

//编写一个函数，每隔1秒输出“hello world”
func test(){   
    for i:=1;i<10 ;i++  {   
        fmt.Println("test() hello world"+strconv.Itoa(i))     
        
        time.Sleep(time.Second)  //休眠一秒  
    }
}

func main(){  
    go test()//开启一个协程    

    for i:=1;i<10 ;i++  {    
        fmt.Println("main() hello world"+strconv.Itoa(i))      
        
        time.Sleep(time.Second)     //休眠一秒  
    }
}
```
输出：
```go
main() hello world1
test() hello world1
main() hello world2
test() hello world2
test() hello world3
main() hello world3
main() hello world4
test() hello world4
test() hello world5
main() hello world5
test() hello world6
main() hello world6
main() hello world7
test() hello world7
test() hello world8
main() hello world8
main() hello world9
test() hello world9
```
由输出看出，**主线程main和协程test同时执行**

* 主线程和协程执行流程图
![goroutine1](./goroutine1.jpg)

* 程序开始执行，main主线程开始执行
* go test()开启协程，**主线程main和协程test同时执行**
* 程序退出以**主线程**为主：
       1） 如果主线程退出了，则协程即使还没有执行完毕也会退出
       2）当然协程可以在主线程没有退出前，就执行完毕退出协程
       
4、总结

* 1）**主线程**是一个**物理线程**，**直接作用在CPU上**。是重量级的，非常耗费CPU资源。

* 2）**协程从主线程开启的**，是轻量级的线程，是**逻辑态的**。对资源消耗相对小

* 3）Go的协程机制是重要特点，可以轻松的开启上万个协程。（其他编程语言的开发机制一般是基于线程的，开启过多的线程，资源消耗大）

#### 2、goroutine的调度模型
1、MPG模式基本介绍

* M：操作系统的主线程（是物理线程，真正干活的人）
* P：协程执行需要的上下文环境（运行时需要的资源和运行时的状态）
* G：协程（逻辑态的）

2、MPG模式运行的状态

1）MPG模式运行的状态1
![goroutine2](./goroutine2.jpg)


2）MPG模式运行的状态2
![goroutine3](./goroutine3.jpg)

#### 3、设置运行CPU数目
```go
package main
import "runtime"

func main() { 
    //runtime.NumCPU()   查询系统的CPU数目
    cpuNum := runtime.NumCPU()   
    println("cpuNum=",cpuNum)     

    //设置使用多个CPU     
    runtime.GOMAXPROCS(cpuNum-1)     
    println("ok")
}
```
### 2、管道channel
案例：计算1-20的各个数的阶乘，并且把各个数放入map中并打印，使用goroutine完成

思路：

* 1、编写一个函数，计算各个数的阶乘并放入map中
* 2、启动多个协程，统计的结果放入map中
* 3、map应该做全局的

解法一：使用全局变量加锁同步
```go
package main

import (     
    "fmt"   
    "sync"   
    "time"
)

var(  
    myMap =make(map[int]int,10)
    //全局资源不加锁，会发生资源竞争，同时写会报错   

    //可以声明一个全局互斥锁解决    
    //sync是一个包，synchornized 同步    
    //Mutex 互斥      
    lock sync.Mutex
)

//test函数计算n的阶乘，将结果放入myMap中
func test(n int){    
    res:=1    
    for i:=1;i<=n;i++{      
    res*=i  
    }    

    //写入map前加锁    
    lock.Lock()     

    //把结果放入myMap     
    myMap[n]=res    

    //写完map后解锁    
    lock.Unlock()
}

func main() {  
    //这里开启20个协程，
    //20个协程同时向map写数据，会发生 并发map写 错误    
    for i:=1;i<=20;i++{         
        go test(i)    
    }   

    //休眠5秒钟（人为估算），让主线程等待所有的协程执行完   
    time.Sleep(5*time.Second)   
    //如果不休眠，可能main主线程已经结束退出，但是test协程还没写入map     

    //互斥性资源读写都要加锁     
    lock.Lock()     

    //输出结果，对map进行读操作     
    for i,v :=range myMap{           
        fmt.Printf("map[%d]=%d\n",i,v)   
    }    

    lock.Unlock()}
```
输出：
```go
map[17]=355687428096000
map[16]=20922789888000
map[20]=2432902008176640000
map[9]=362880
map[10]=3628800
map[12]=479001600
map[1]=1
map[3]=6
map[19]=121645100408832000
map[11]=39916800
map[15]=1307674368000
map[18]=6402373705728000
map[8]=40320
map[5]=120
map[7]=5040
map[14]=87178291200
map[13]=6227020800
map[2]=2
map[4]=24
map[6]=720
```
因为map的key是无序的，所以未按递增顺序执行，而且并发执行的顺序也可能不是递增得到结果的

上面的解法中

* 主线程等待所有goroutine全部完成时间很难确定，这里设置为5秒，是为估算
* 如果主线程休眠时间长了，会加长等待时间；如果等待时间短了，可能还有goroutine处于工作状态（没有执行完），这时也会随主线程的退出二销毁
* 通过全局变量加锁同步来实现协程间通讯，并不利于多个协程对全局变量的读写操作

综上，我们引出一种新的通讯机制channel

解法二：使用channel

#### 1、基本介绍
1、channel本质上就是一个数据结构-队列

2、数据先进先出

3、**线程安全**，多个goroutine访问时，自己不需要再加锁，即：**channel本身就是线程安全的**

4、**channel是有类型的**，一个string的channel只能存放string类型数据
![channel1](./channel1.jpg)

#### 2、基本使用
##### 1、定义/声明
```go
var 变量名 chan 数据类型

//例
var intChan chan int //intChan类型为int，只能存放int型数据

var mapChan chan map[int]string
//mapChan类型为map[int]string，只能存放map[int]string型数据

var perChan chan Person

var perChan2 chan *Person
```
2、注意

* channel是**引用类型**
* **channel必须初始化后才能写入数据，即make后才能使用**
* channel是由类型的，如intChan类型为int，只能存放int型数据

##### 3、管道的初始化、从管道读写数据以及注意事项
```go
package main
import "fmt"

func main() {   
    //1、创建一个可以存放3个int的管道   
    var intChan chan int    
    intChan=make(chan int,3)  //channel make后才能使用

    //2、intChan是什么     
    fmt.Printf("intChan的值=%v intChan本身的地址=%v\n",intChan,&intChan)   
    //输出：intChan的值=0xc000092080 
    //intChan本身的地址=0xc00008c018      
    //可以看出intChan的值为一个地址，所以channel是引用类型。    

    //3、向管道写入数据     
    intChan<-10//直接写入常量   
    num:=211     
    intChan<-num//写入变量    

    //4、看看管道的长度和容量    
    //容量是make时传入的，这里传入的是3，容量不能自动增长，和slice、map不一样    
    fmt.Printf("channel len=%v cap=%v\n",len(intChan),cap(intChan))     
    //channel len=2 cap=3    

    //5、注意：给管道写入数据时不能超过容量     
    intChan<-50      //intChan<-98     
    fmt.Printf("channel len=%v cap=%v\n",len(intChan),cap(intChan))   
    //fatal error: all goroutines are asleep - deadlock!   
    //报告死锁deadlock错误     

    //6、从管道中读取数据     
    var num2 int     
    num2 = <-intChan  
    fmt.Println("num2=",num2)//num2= 10，从队列头开始取     
    fmt.Printf("channel len=%v cap=%v\n",len(intChan),cap(intChan))    
    //channel len=2 cap=3     

    //7、在没有使用协程的情况下，如果管道中的数据已经全部取出，再取数据就会报告死锁deadlock错误     
    num3 := <-intChan    
    num4 := <-intChan     
    num5 := <-intChan   
    fmt.Println("num3=",num3,"num4=",num4,"num5=",num5)   
    //fatal error: all goroutines are asleep - deadlock!
}
```
注意事项

* channel中只能存放指定的数据类型
* channel的数据放满后，就不能再放入了，否则报告死锁deadlock错误
* 当 channel的数据放满后，如果从channel中取出数据，可以再次放入数据
* 在没有使用协程的情况下，如果channel中的数据取完了，再次取数据，会报告死锁deadlock错误

一个例子，当管道是空接口interface{}类型时，可以存放任意数据类型的值，取出管道中的值对象的的字段值时，需要类型断言
```go
package main
import "fmt"

type Cat struct { 
    Name string     
    age int
}

func main() {  
    allChan:=make(chan interface{},3)//定义一个空接口类型，容量为3的管道allChan      

    allChan<-10//往管道写入int类型的值     
    allChan<-"tom"//往管道写入string类型的值    

    cat:=Cat{"小花猫",4}//实例化一个Cat    
    allChan<-cat////往管道写入Cat类型的值    

    //想要取出管道中的第三个值，需要丢弃第一个和第二个    
    <-allChan    
    <-allChan    

    //取到第三个   
    newCat:=<-allChan   

    //newCat从编译层面看是空接口类型,接口类型本身没有字段    
    //但在运行时可动态指向结构体Cat，
    //下面写法语法上没错，运行时可动态指向Cat     
    fmt.Printf("newCat的类型=%T newCat的值=%v\n",newCat,newCat)   
    //输出：newCat的类型=main.Cat newCat的值={小花猫 4}

    //newCat从编译层面看是空接口类型，接口类型本身没有字段，
    //直接写编译不通过，需要类型断言      
    aNewCat:=newCat.(Cat)     
    fmt.Printf("newCat.Name=%v",aNewCat.Name)
    //输出：newCat.Name=小花猫
 }
```

##### 4、管道的遍历和关闭

 1、管道的关闭

使用内置函数**close()可以关闭管道**，当管道关闭后，就不能再向管道写数据了，但是仍然可以从管道中读取数据
```go
package main
import "fmt"

func main() {
    intChan:=make(chan int,3)  
    intChan<-100   
    intChan<-200    

    //1、关闭管道    
    close(intChan)    

    //2、关闭管道后，不能再写入数据   
    //intChan<-300    
    //panic: send on closed channel     
    //向一个关闭的通道中发送数据，panic终止程序    

    //3、关闭管道后，可以再读取数据   
    n1:=<-intChan    
    fmt.Println("n1=",n1)//n1= 100
}
```
2、管道的遍历
channel支持for-range的方式进行遍历，有两个细节注意：

* 在遍历时，如果channel没有关闭，则会出现死锁deadlock错误
* 在遍历时，如果channel已经关闭，则会正常遍历数据，遍历完后，就会退出遍历

在遍历时不能使用普通for循环遍历，因为管道的长度（len(intChan)）是随数据出管道-1动态变化的
```go
package main
import "fmt"

func main() {   
    intChan:=make(chan int,100)    

    //循环往管道中写入100个数据      
    for i:=0;i<100 ;i++  {       
        intChan<-i*2   
    }    

    //遍历，一定要先关闭管道,如果不关闭管道，会出现死锁deadlock错误     
    close(intChan)    

    //管道没有下标，for-range只返回一个值    
    for v:= range intChan{         
        fmt.Println("v=",v)     
    }
}
```

### 3、goroutine和管道channel结合使用
#### 1、案例：
案例：
goroutine和管道channel协同工作
1）开启一个wiiteData协程，向管道写入50个整数
2）开启一个readData协程，从管道中读取wiiteData写入的数据
3）注意：wiiteData和readData操作的是同一个管道
4）主线程需要等待wiiteData和readData协程完成工作后才能退出
思路图解：
![goandchan1](./goandchan1.jpg)
```go
package main
import "fmt"

//writeData
func writeData(intChan chan int){ 
    for i:=1;i<=50;i++  {   
        intChan<-i         
        fmt.Println("writeData ",i)    
    }      

    close(intChan)//关闭intChan，后面intChan仍然可读
}

//readData
func readData(intChan chan int,exitChan chan bool){      
    for{         
        v,ok:=<-intChan      
        if !ok {       
          break     
         }         
        fmt.Printf("readData 读到数据=%v\n",v)   
    }    

    //readData读取数据完后，    
    //往exitChan写入数据true，告知主线程readData执行完毕    
    exitChan<-true   
    close(exitChan)
}

func main() {    
    //创建两个管道  
    intChan:=make(chan int,50)  
    exitChan:=make(chan bool,1)  //该管道用于等到intChan管道的读取协程工作完毕后，往exitChan写入数据，标识主线程可退出
    
    go writeData(intChan) //向管道写入50个整数
    go readData(intChan,exitChan)   //从管道中读取wiiteData写入的数据
    
    for{      
        _,ok:=<-exitChan    
        if !ok {            
             break       
        }   
    }
}
```
#### 2、管道阻塞机制
如果**只向管道里写数据，而没有读取**，就会出现**阻塞而死锁**deadlock。
注意：如果**有向管道读取数据，但读取比写数据慢得多**，也不会发生死锁，只要编译器检测到数据在管道中是流动的，即有读取也有写入，那么就不会发生死锁

下面例子中，intChan容量改为10
```go
package main
import "fmt"

//writeData
func writeData(intChan chan int){ 
    for i:=1;i<=50;i++  {   
        intChan<-i     //数据一下就放入，但下面readData读取慢慢读
    
        fmt.Println("writeData ",i)    
    }      

    close(intChan)//关闭intChan，后面intChan仍然可读
}

//readData
func readData(intChan chan int,exitChan chan bool){      
    for{         
        v,ok:=<-intChan      
        if !ok {       
          break     
         }         
       time.Sleep(time.Second)//慢慢读，每隔一秒读一次
       
      fmt.Printf("readData 读到数据=%v\n",v)   
    }    

    //readData读取数据完后，    
    //往exitChan写入数据true，告知主线程readData执行完毕    
    exitChan<-true   
    close(exitChan)
}

func main() {    
    //创建两个管道  
    intChan:=make(chan int,10)//容量10
    exitChan:=make(chan bool,1)  //该管道用于等到intChan管道的读取协程工作完毕后，往exitChan写入数据，标识主线程可退出
    
    go writeData(intChan) //向管道写入50个整数
    go readData(intChan,exitChan)   //从管道中读取wiiteData写入的数据
    
    for{      
        _,ok:=<-exitChan    
        if !ok {            
             break       
        }   
    }
}
```
输出：
```go
writeData  1
writeData  2
writeData  3
writeData  4
writeData  5
writeData  6
writeData  7
writeData  8
writeData  9
writeData  10
writeData  11
readData 读到数据=1
writeData  12
readData 读到数据=2
writeData  13
readData 读到数据=3
writeData  14
readData 读到数据=4
writeData  15
readData 读到数据=5
writeData  16
readData 读到数据=6
writeData  17
readData 读到数据=7
writeData  18
readData 读到数据=8
writeData  19
readData 读到数据=9
writeData  20
readData 读到数据=10
writeData  21
readData 读到数据=11
writeData  22
readData 读到数据=12
writeData  23
readData 读到数据=13
writeData  24
readData 读到数据=14
writeData  25
readData 读到数据=15
writeData  26
readData 读到数据=16
writeData  27
readData 读到数据=17
writeData  28
readData 读到数据=18
writeData  29
readData 读到数据=19
writeData  30
readData 读到数据=20
writeData  31
readData 读到数据=21
writeData  32
readData 读到数据=22
writeData  33
readData 读到数据=23
writeData  34
readData 读到数据=24
writeData  35
readData 读到数据=25
writeData  36
readData 读到数据=26
writeData  37
readData 读到数据=27
writeData  38
readData 读到数据=28
writeData  39
readData 读到数据=29
writeData  40
readData 读到数据=30
writeData  41
readData 读到数据=31
writeData  42
readData 读到数据=32
writeData  43
readData 读到数据=33
writeData  44
readData 读到数据=34
writeData  45
readData 读到数据=35
writeData  46
readData 读到数据=36
writeData  47
readData 读到数据=37
writeData  48
readData 读到数据=38
writeData  49
readData 读到数据=39
writeData  50
readData 读到数据=40
readData 读到数据=41
readData 读到数据=42
readData 读到数据=43
readData 读到数据=44
readData 读到数据=45
readData 读到数据=46
readData 读到数据=47
readData 读到数据=48
readData 读到数据=49
readData 读到数据=50
```
因为管道容量为10，所以写到11的时候，需要等待数据被读出才能写入，所以写数据12-50与读取是交错的，读取数据每隔一秒读取一次。最后数据全部写完，只需要读取数据。

#### 3、细节
1、在默认情况下，管道是双向的，即可读可写
2、管道可以声明为只读或只写
```go
//管道声明为只写
var chan1 chan<- int
chan1 = make(chan int,3)
chan1 <- 1 //ok
num1 := <-chan1 //error，无效的操作

//管道声明为只读
var chan2 <-chan int
chan2 = make(chan int,3)
num2 := <-chan2 //ok
chan2 <- 2 //error
```
管道声明为只读或只写的应用场景：

* 先声明一个双向管道intChan
* 声明两个函数，一个往intChan只写数据的send函数，一个往intChan只读数据的recv函数，防止误操作
* 可以将双向管道intChan作为实参，传给只写数据的send函数，send函数形参为**ch chan<- int**
* 可以将双向管道intChan作为实参，传给只读数据的recv函数，re函数形参为**ch <-chan int**
* 上面所述中，将双向管道作为实参传给单向管道（只读或只写）并不会报错。
* 管道的双向和单向只是管道的性质，但是管道的类型都是chan int，所以不会报错

3、使用**select**可以解决**从管道取数据的阻塞问题**
```go
package main
import (    
"fmt"
)

func main() {   
    //使用select可以解决从管道取数据的阻塞问题     

    //1、定义一个管道 10个数据 int     
    intChan := make(chan int,10)    
    for i:=0;i<10;i++  {     
         intChan<-i     
    }     

    //2、定义一个管道 5个数据，string     
    stringChan := make(chan string,5)    
    for i:=0;i<5 ;i++  {   
         stringChan<-"hello"+fmt.Sprintf("%d",i)  
    }   

    //传统方法在遍历管道时，如果不关闭会阻塞并死锁deadlock   
    //但在实际开发中，我们往往不确定关闭管道的时机    

    //由此我们使用select解决      
    for{          
        select {          
        //重点：这里如果intChan一直没有关闭，不会一直阻塞而死锁，                
        //会自动到下一个case匹配         
            case v:=<-intChan:              
            fmt.Printf("从intChan读取了数据%d\n",v)         
            case v:=<-stringChan:                   
            fmt.Printf("从stringChan读取了数据%s\n",v)              
            default:                     
            fmt.Printf("都取不到了，加入处理逻辑\n")                    
            return        
        }    
    }
}
```
输出：
```go
从stringChan读取了数据hello0
从intChan读取了数据0
从stringChan读取了数据hello1
从stringChan读取了数据hello2
从stringChan读取了数据hello3
从intChan读取了数据1
从intChan读取了数据2
从stringChan读取了数据hello4
从intChan读取了数据3
从intChan读取了数据4
从intChan读取了数据5
从intChan读取了数据6
从intChan读取了数据7
从intChan读取了数据8
从intChan读取了数据9
都取不到了，加入处理逻辑
```

4、**goroutine中使用recover，解决协程中出现panic**，导致程序崩溃问题

* 如果我们起了一个协程，但是这个协程出现了panic，如果我们没有捕获这个panic，就会造成成哥程序的崩溃，

* 这时可以在该协程中使用recover来捕获panic进行处理。这样即使这个协程发生问题，主线程仍然不受影响，继续执行。

```go
package main
import (    
    "fmt"   
    "time"
)

//函数sayHello
func sayHello(){  
    for i:=0;i<10 ;i++  {           
        time.Sleep(time.Second)    
        fmt.Println("hello world")  
    }
}

//函数test
func test(){   
    //这里使用defer+recover解决panic终止程序    
    defer func() {           
    //捕获test抛出的panic         
        if err:=recover();err!=nil {            
              fmt.Println("test() 发生错误")   
        }    
    }()     

    //定义一个map  
    var myMap map[int]string     
    myMap[0]="golang"//空map直接赋值，报错
}

func main() {   
    go sayHello()    
    go test()   //这个协程会panic 使用defer+recover解决

    for i:=0;i<10 ;i++  {       
        time.Sleep(time.Second)       
        fmt.Println("main() ok=",i)   
    }
}
```
输出：
```go
test() 发生错误
hello world
main() ok= 0
main() ok= 1
hello world
hello world
main() ok= 2
hello world
main() ok= 3
hello world
main() ok= 4
main() ok= 5
hello world
hello world
main() ok= 6
main() ok= 7
hello world
hello world
main() ok= 8
main() ok= 9
```
由输出看出，test协程出错，但是主线程和sayHello协程继续执行

### 4、反射
#### 1、基本介绍
1、反射的作用：

* 1）、反射可以在运行时动态获取变量的各种信息，如变量的类型（type）、类别（kind）

* 2）、如果是结构体变量，还可以获取到结构体本身的信息（包括结构体的字段、方法）

* 3）、通过反射，可以修改变量的值，可以调用关联的方法

* 4）、使用反射，需要import（“reflect”）

reflect实现了运行时反射，允许程序操作任意类型的对象，典型用法是用静态类型interface{}保存一个值，通过**调用TypeOf获取类型信息**，该函数返回一个Type类型值，**调用ValueOf函数返回一个value类型值**，该值代表运行时数据，Zero接受一个Type类型参数并返回该类型零值的value类型值。

* 5）、反射示意图

![reflect1](./reflect1.png)

**reflect.Type是一个接口**，定义了非常多方法，通过这些方法可以反向操作变量，获取变量的各种信息

**reflect.Value是一个结构体**，包含了非常多方法，可通过Type()方法将Value转换为Type、返回变量的字段和方法等等

* 6）、变量、interface{}和reflect.Value是可以相互转换的
![reflect2](./reflect2.png)

####   2、案例
1、对基本数据类型、interface{}、reflect.Value进行反射的基本操作
```go
package main
import (     
    "fmt"   
    "reflect"
)

//反射
func reflectTest01(b interface{}){  
    //通过反射获取传入变量的type、kind、值     

    //1、先获取到reflect.Type     
    rTyp := reflect.TypeOf(b)     
    fmt.Println("rTyp=",rTyp)     

    //2、获取到reflect.Value     
    rVal := reflect.ValueOf(b)//rVal=100      
    fmt.Printf("rVal=%v rVal的type=%T\n",rVal,rVal)     
    //n:=rVal+2 
    //error，rVal的类型不是int，是reflect.Value，不能做运算    

    //3、如果要做运算，如下      
    n:=rVal.Int()+2     
    fmt.Println("n=",n)    

    //4、将rval转成interface{}      
    iv:=rVal.Interface()     

    //5、将interface{}通过断言转成int(对应的类型)    
    num2:=iv.(int)     
    fmt.Printf("num2=%v num2的type=%T\n",num2,num2)
}

func main() {    
    //1、定义一个int     
    var num int = 100     
    
    //2、反射
    reflectTest01(num)
}
```
输出：
```go
rTyp= int
rVal=100 rVal的type=reflect.Value
n= 102
num2=100 num2的type=int
```

2、对结构体、interface{}、reflect.Value进行反射的基本操作
```go
package main
import (
    "fmt"
    "reflect"
)

type Student struct {  
    Name string     
    age int
}

//对结构体反射
func reflectTest02(b interface{}){  
    //通过反射获取传入变量的type、kind、值     
    //1、先获取到reflect.Type   
    rTyp := reflect.TypeOf(b)    
    fmt.Println("rTyp=",rTyp)   

    //2、获取到reflect.Value     
    rVal := reflect.ValueOf(b)   
    fmt.Printf("rVal=%v rVal的type=%T\n",rVal,rVal)    
    
    //3、获取变量对应的kind，两种方式等价
    //(1)通过获取到的reflect.Type来获取 =》rTyp.Kind()
    kind1:=rTyp.Kind()
    //(2)通过获取到的reflect.Value来获取 =》rVal.Kind()
   kind2:=rVal.Kind()
   fmt.Printf("kind1=%v kind2=%v\n",kind1,kind2)

    //3、将rval转成interface{}     
    iv:=rVal.Interface()     
    fmt.Printf("iv=%v iv的type=%T\n",iv,iv)  
    //输出：iv={tom 20} iv的type=main.Student   

    //4、iv的type=main.Student，但是通过iv取字段会报错     
    //iv.Name     
    //error 因为编译器在编译阶段无法知道iv的类型是main.Student，只有运行时才知道     
    //所以这里直接编译报错    
    //反射是在程序运行时工作的    

    //5、将interface{}通过断言转成int(对应的类型)    
    stu:=iv.(Student)    
    if ok {     
        fmt.Printf("stu=%v stu的type=%T\n",stu,stu)    
        fmt.Printf("stu.Name=%v stu.Age=%v\n",stu.Name,stu.Age)
    }
}

func main() {  
    //1、定义一个student实例   
    stu:=Student{"tom",20}     
    //2、反射   
    reflectTest02(stu)
}
```
输出：
```go
rTyp= main.Student
rVal={tom 20} rVal的type=reflect.Value
kind1=struct kind2=struct
iv={tom 20} iv的type=main.Student
stu={tom 20} stu的type=main.Student
stu.Name=tom stu.Age=20
```
####   3、注意事项
1、reflect.Value.Kind()，获取变量的类别，返回一个常量。type是一个大范畴，kind在type上细分（如type只返回int，kind返回具体的int32，int64的常量定义的值等）
在上面的例子中有：
```go
//3、获取变量对应的kind，两种方式等价
    //(1)通过获取到的reflect.Type来获取 =》rTyp.Kind()
    kind1:=rTyp.Kind()
    //(2)通过获取到的reflect.Value来获取 =》rVal.Kind()
   kind2:=rVal.Kind()
   fmt.Printf("kind1=%v kind2=%v\n",kind1,kind2)
```
输出：
```go
kind1=struct kind2=struct
```
2、Type和Kind的区别：Type是类型，Kind是类别，**Type和Kind可能相同也可能不同**

* var num int = 10 num的Type是int，Kind也是int
* var stu Student   stu的Type是**包名.Student**，Kind是struct

3、变量、interface{}和reflect.Value是可以相互转换的

4、使用反射的方式来获取变量的值（并返回对应的类型），要求数据类型匹配，如x是int，那么就应该用reflect.ValueOf(x).Int()，不能使用其他，否则panic

5、通过反射修改变量的值，注意当使用SetXXX方法来修改变量的值，需要通过变量对应的指针来修改，这时需要使用reflect.Value.Elem()方法

* func (v Value) SetXXX(x XXX)
```go
 func (v Value) SetXXX(x XXX) 
//SetXXX方法是和Value类型绑定的方法，不能用指针类型去调用它，只能把指针类型再转为Value类型，使用Elem函数    
```

* func (v Value) Elem() Value   
```go
func (v Value) Elem() Value
 //重点：Elem返回v持有的接口保管的值的Value类型封装，或持有的指针指向的值的Value类型封装     
```
代码：
```go
package main
import (    
"fmt"    
"reflect"
)

//修改反射变量的值    
func reflectTest01(b interface{}){    
    //1、获取到reflect.Value  
    rVal := reflect.ValueOf(b)//rVal=100    
    fmt.Printf("rVal=%v rVal的type=%T\n",rVal,rVal)  
    //main函数中调用函数时传的是地址，rVal的值为一个地址

    //2、获取rVal的Kind    
    rKind:=rVal.Kind()  
    fmt.Printf("rVal Kind=%v\n",rKind)   
    //输出：rVal Kind=ptr，
    //rVal的Kind是指针，因为main函数中调用函数时传的是地址     

    //3、通过SetXXX修改变量的值    
    //rVal.SetInt(20) //error   
    
    //func (v Value) SetXXX(x XXX)，  
    //SetXXX方法是和Value类型绑定的方法，不能用指针类型去调用它    
    //只能把指针类型再转为Value类型，使用Elem函数    
    
    //func (v Value) Elem() Value   
    //Elem返回v持有的接口保管的值的Value类型封装，或持有的指针指向的值的Value类型封装     

    //4、通过Elem()获取到指针指向的值，再通过SetXXX修改变量的值     
    rVal.Elem().SetInt(20)    //值修改为20
}

func main() {     
    //1、定义一个int    
    var num int = 100     
    //2、反射    
    reflectTest01(&num)//修改变量num的值，所以要穿num的地址
    
    //3、打印修改后的num
    fmt.Println("num=",num)
}
```
输出：
```go
rVal=0xc000060058 rVal的type=reflect.Value
rVal Kind=ptr
num= 20
```