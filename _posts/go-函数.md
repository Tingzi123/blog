---
title: 'go:函数'
date: 2020-06-11 15:19:21
tags:
---
### 1、函数
#### 1、函数调用底层机制
有如下示例
下图中的内存空间划分均为逻辑划分
1. 从main函数入口开始执行程序
![func1](./func1.jpg)
在内存中会为main函数在栈上开辟一个空间，并为变量n1赋值10

2、调用函数test()
![func2](./func2.jpg)
逻辑上内存会为函数test()在栈上另外开辟一个空间（逻辑上），用于存放test（）函数中的数据

3、调用函数test()，并传入参数n1
![func3](./func3.jpg)
test（）函数接受main（）函数传过来的参数n1，**在test栈区中另开辟一个空间存放传过来的参数**，此时，**main栈区和test栈区的n1已经没有关系了（引用类型除外）**，所以test栈区中n1的改变不影响main栈区中的n1

4、test()函数执行n1=n1+1
![func4](./func4.jpg)
test栈区中n1的改变不影响main栈区中的n1

5、test()函数执行打印n1语句，终端输出test栈区中的n1
![func5](./func5.jpg)

6、test（）函数执行完之后，内存回收test栈区，**test栈区中数据全部被清除**，程序回到main（）函数调用test（）函数的地方
![func6](./func6.jpg)

7、main()函数执行打印n1语句，终端输出main栈区中的n1
![func7](./func7.jpg)

8、main（）函数执行完之后，内存回收main栈区，**main栈区中数据全部被清除**，程序结束
![func8](./func8.jpg)

9、**总结**

* 在调用一个函数时，会为该函数分配一个新的栈空间，编译器会通过自身的处理让这个新的空间和其他的栈区空间区分开来
* 在每个函数对应的栈中，数据空间是独立的，不会混淆
* 当一个函数调用（执行）完毕后，程序会销毁这个函数对应的栈空间

10、注意事项

* Go不支持传统的函数重载


* Go中，**函数本身也是一种数据类型**。可以赋值给一个变量，则该变量就是一个函数类型的变量，**通过该变量可以对函数调用**
```go
func getSum(n1 int,n2 int) int {
       return n1+n2
}

func main(){
    a:=getSum   //把函数getSum赋给变量a
    fmt.Println("a的类型是%T,getSum的类型是%T\n",a,getSum)
    
    res := a(10,40)//等价于 res := getSum(10,40)
    fmt.Println("res=",res)
}
```

* 因为函数是一种数据类型，所以在Go中，**函数可以作为形参并调用**。
```go

\
```
* Go支持对函数返回值命名
```go
func getSumAndSub(n1 int,n2 int) (sum int, sub int) {
       ///返回值命名sum和sub
       sub = n1 - n2
       sum = n1 + n2
       //为sum和sub赋值，顺序无所谓，可与返回值列表顺序不同
       return      
       //等价于return sum , sub ，由于已为返回值命名sum和sub，函数体中已为sum和sub赋值，return时可省略不写返回值，再写上反而显得冗余  
}
```

* Go支持可变参数
```go
 //支持0到多个参数
func sum(args... int) sum int {
     
}

 //支持1到多个参数
func sum(n1 int , args... int) sum int {
     
}
```
args是切片slice，通过args[index]可以访问多个值

#### 2、init函数
1、**每一个源文件中都可以包含init函数**，该函数会在main函数执行之前，被Go运行框架调用，即init函数在main函数之前被调用。该函数通常用作初始化工作

2、如果一个文件同时包含全局变量定义，init函数和main函数，则执行的流程是：**全局变量定义->init函数->main函数**

3、如果包含main函数的包里import了其他包，执行流程如下
![func9](./func9.jpg)

#### 3、匿名函数
1、如果某个函数我们只是希望使用一次，可以考虑使用匿名函数，匿名函数也可以实现多次调用

2、匿名函数的三种使用方式
* 在定义匿名函数时就直接调用，且只能调用一次

```go
func main(){
    res1 := func(n1 int, n2 int) int{
        return n1 + n2
     }(10,20)
     //在定义匿名函数的同时就调用它
     
      fmt.Println("res1=",res1)
}
```
通过在紧跟匿名函数定义的后面传入参数调用
![func10](./func10.png)

* 将匿名函数赋给一个变量（函数变量），再通过该变量来调用匿名函数

```go
func main(){
    //将匿名函数func(n1 int, n2 int) int 赋值给变量a
    //则a的数据类型就是函数类型，可以通过变量a完成调用
    a := func(n1 int, n2 int) int{
        return n1 + n2
     }
    
    res2 := a(10,20)
    res3 := a(90,20)
    
     fmt.Println("res2=",res2)
     fmt.Println("res3=",res3)
}

```
* 将匿名函数赋给一个全局变量，则这个匿名函数成为一个全局匿名函数，可以在整个程序有效

```go
    var(
        Func1 = func(n1 int, n2 int) int{
             return n1 + n2
         }
         //这时func1即是一个全局匿名函数
    )

    func main(){
        res4 := Func1(4,9)
        fmt.Println("res4=",res4)
    }
```
#### 4、闭包
1、闭包：就是**一个函数**和**与其相关的引用环境**组合的一个整体（实体）
```go
    //累加器
    func AddUpper() func(int) int {
        var n int = 10
        return func(x int) int {
            n = n + x
            return n
        }
    }

    func main(){
        f := AddUpper()
        fmt.Println(f(1)) //输出11
        fmt.Println(f(2)) //输出11+2=13
        fmt.Println(f(3)) //输出13+3=16
    }
```
对上面代码的说明

* AddUpper是一个函数，处于外层，返回数据类型时func(int) int
* **闭包**的说明

![func11](./func11.png)

在上图红框中可以看到，return返回一个匿名函数，处于内层，这个匿名函数**引用到外层函数的变量n**，因此这个**匿名函数和变量n形成一个整体**，构成**闭包**

* 可以理解成：闭包是一个类，函数是操作，变量n是字段。函数和它使用到的字段n构成闭包

* 在main函数中，外层函数AddUpper只被调用一次，所以AddUpper函数中的n只被初始化一次

* 当我们反复调用内层函数f时，因为外层函数中的变量n只初始化一次，因此每次调用进行累计，而不会重新初始化

* 闭包的关键，是要分析出返回的内层函数它所引用到的外层函数中的变量，因为函数和它所引用到的变量构成闭包

#### 5、函数中的defer
1、为什么需要defer
    在函数中，经常需要创建资源（如数据库连接、文件句柄、锁等），**为了在函数执行完毕后，及时的释放资源**，Go提供defer（延迟机制）
    
 2、defer使用案例
 ```go
    func sum(n1 int,n2 int) int {
        //当执行到defer时，暂时不执行defer后的语句，会将defer后的语句压入独立的栈（这个栈和main函数及sum函数的栈不一样，这里称为defer栈）中
        
        //当函数执行完毕后，再从defer栈中，按先入后出顺序出栈执行
       defer fmt.Println("n1=",n1)//3 n1=10
       defer fmt.Println("n2=",n2)//2 n2=20
       
       res := n1+n2
       fmt.Println("n1+n2=",res)//1 n1+n2=30
       return res
    }

    func main(){
        res := sum(10,20)
        fmt.Println("res=",res)//4 res=30
    }
```
输出顺序：
 ```go
n1+n2=30
n2=20
n1=10
res=30
```

 3、当执行到defer时，暂时不执行defer后的语句，会将defer后的语句压入独立的栈中，然后执行函数的下一条语句
 
 4、当函数执行完毕后，再从defer栈中，按先入后出顺序出栈执行
 
 5、将defer后的语句放入栈时，也会**将相关的值拷贝同时入栈**
  ```go
    func sum(n1 int,n2 int) int {
       defer fmt.Println("n1=",n1)//defer  3 n1=10
       defer fmt.Println("n2=",n2)//defer  2 n2=20
       
       //增加两条语句
      n1++ //n1=11
      n2++ //n2=21
      //这里n1、n2自增后，并不影响defer栈中的值，因为defer之后的语句入栈时，同时将值拷贝入栈，所以defer中的n1、n2值仍是入栈时候的值
      
       res := n1+n2
       fmt.Println("n1+n2=",res)//1 n1+n2=32
       return res
    }

    func main(){
        res := sum(10,20)
        fmt.Println("res=",res)//4 res=32
    }
```
输出顺序：
 ```go
n1+n2=32
n2=20
n1=10
res=32
```

#### 6、函数参数传递的方式
1、两种传递方式

* **值传递**
* **引用传递**

不管是值传递还是引用传递，**传递给函数的都是变量的副本**。不同的是：**值传递**传递的是**值的拷贝**，**引用传递**传递的是**地址的拷贝**。

一般来说，地址拷贝效率高，因为数据量小，而值拷贝的效率由传递的数据大小决定，数据越大，效率越低。

2、值类型和引用类型

* 值类型：**基本数据类型（int系列、float系列、bool、string）、数组、结构体**
* 引用类型：**指针、slice切片、map、管道channel、interface**等


#### 7、字符串中常用系统函数
1、统计字符串的长度，按**字节**：**len(str)**
是一个内置（内建）函数，不属于任何包，可直接使用

2、字符串遍历，同时处理 有中文的问题：**r := []rune(str)**
遍历字符串时，可先将string类型的字符串str转成rune的切片：[]rune(str)，再进行遍历，因为string类型按字节遍历，[]rune按字符遍历，一个中文占三个字节。（for循环按字节遍历，for ... range 按字符遍历）

3、字符串转整数：**n,err := strconv.Atoi("12")**

4、整数转字符串：**str = strconv.Itoa(12345)**

5、字符串转[]byte：**var bytes = []byte("hello go")**

6、[]byte转字符串：**str = string([]byte{97,98,99})**

7、十进制转二进制、八进制、十六进制

* 十进制转二进制:

**str = strconv.FormatInt(123,2)**

* 十进制转八进制:

str = strconv.FormatInt(123,**8**)

* 十进制转十六进制:

str = strconv.FormatInt(123,**16**)

8、查找子串是否在指定的串中：**strings.Contains("seafood","foo")**//true

9、统计一个字符串中有几个指定的子串：strings.Count("ceheese","e")//4

10、不区分大小写的字符串比较（用“==”比较区分大小写）：**strings.EqualFold("abc","Abc")**//true

11、返回子串在字符串中第一次出现的index值，如果没有返回-1：**strings.Index("NLT_abc","abc")**//4  下标从0开始索引

12、返回子串在字符串中最后一次出现的index值，如果没有返回-1：**strings.LastIndex("go go hello","go")**//3  下标从0开始索引

13、将指定的子串替换成另外一个子串：**strings.Replace("go go hello","go","go语言"，n)**
n可以指定你希望替换几个，n=-1表示全部替换

14、按照指定的某个字符为分割标识，将一个字符串拆分成字符串数组：**strings.Split("hello,world,ok",",")**

15、将字符串的字母进行大小写转换：

大写转小写
```go
strings.ToLower("Go")//go
```

小写转大写
```go
strings.ToUpper("Go")//GO
```

16、将字符串的左右两边空格去掉：**strings.TrimSpace("  go hello    ")**

17、将字符串左右两边指定的字符去掉：**strings.Trim("! hello!"," !")**
//将【hello】左右两边的！和空格去掉

18、将字符串左边指定的字符去掉：**strings.TrimLeft("! hello!"," !")**
//将【hello】左边的！和空格去掉

19、将字符串右边指定的字符去掉：**strings.TrimRight("! hello!"," !")**
//将【hello】右边的！和空格去掉

20、判断字符串是否以指定的字符串开头：
```go
strings.HasPrefix("ftp://192.168.10.1","ftp")//true
```

21、判断字符串是否以指定的字符串结束：**strings.HasSuffix("go_var.jpg","jpg")**//true

#### 8、日期和时间相关函数
使用时间和日期函数需要引入**time包**
1、**time.Time**类型，用于表示时间

2、获取当前时间
**now := time.Now()**//now类型是time.Time

```go
    通过变量now获取年月日、时分秒
    now.Year()
    now.Month()//英文月份
    int（now.Month()）//int强转为数字月份
    now.Day()
    now.Hour()
    now.Minute()
    now.Second()
```
3、格式化日期
##### 1）方式1使用Printf或者Sprintf

```go
    now := time.Now()
    fmt.Printf("当前年月日时分秒 %02d-%02d-%02d %02d-%02d-%02d \n",now.Year(),now.Month(),now.Day(),now.Hour(),now.Minute(),now.Second())
```
或者Sprintf，Sprintf会返回一个字符串
```go
    now := time.Now()
    dateStr := fmt.Sprintf("当前年月日时分秒 %02d-%02d-%02d %02d-%02d-%02d \n",now.Year(),now.Month(),now.Day(),now.Hour(),now.Minute(),now.Second())
    //fmt.Sprintf返回一个时间日期字符串赋给变量dateStr，可对变量dateStr进行操作(输出或保存等)
    fmt.Printf("dateStr=%v\n",dateStr)
```

##### 2）方式2使用Format

```go
    now := time.Now()
    fmt.printf(now.Format("2006/01/02 15:04:05"))//年月日时分秒
    fmt.printf(now.Format("2006/01/02))//年月日
    fmt.printf(now.Format("15:04:05"))//时分秒
    fmt.printf(now.Format("2006"))//年
    fmt.printf(now.Format("01"))//月
    fmt.printf(now.Format("02"))//日
    fmt.printf(now.Format("2006-01-02 15:04:05"))
```
**“2006/01/02 15:04:05”这一串数字是固定的，不能更改**，但格式和组合可以改（传说这串数字是Go开发者脑海中萌生GoLang的时刻）。

4、时间的常量
```go
    const(
        Nanosecond Duration = 1                             //纳秒
        Microsecond             = 1000 * Nanosecond     //微秒
        Millisecond                = 1000 * Microsecond     //毫秒
        Second                     = 1000 * Millsecond      //秒
        Minute                     = 60 * Second            //分钟
        Hour                        = 60 * Minute           //小时
    )
```
常量的作用：在程序中可用于获取指定时间单位的时间，比如想得到100毫秒 100 * time.Millisecond

可用于休眠
```go
time.Sleep(time.Second)//休眠1秒
time.Sleep(time.Millisecond * 100)//休眠0.1秒
//只能传入整数，不能传小数
//休眠0.1秒不能这么写time.Sleep(time.Second * 0.1)
```

5、时间戳

* unix时间戳
返回1970年1月1日0时0分0秒 到当前时间的秒数

* unixnano时间戳(unix纳秒时间戳)
返回1970年1月1日0时0分0秒 到当前时间的纳秒数

可以用于**获取随机的数字**
```go
//为了每次生成的随机数不一样，需要设定一个种子seed
rand.Seed(time.Now().UnixNano())
num := rand.Intn(100)//0<=n<100
//如果不设置种子，会有一个默认值，但每次随机出来的数都一样
```

#### 9、内置（buildin）函数
1、len()：求长度，比如string、array、slice、map、channel

2、func new
```go
func new(Type) *Type
```
new分配分存，第一个实参为类型而非值，new函数返回值为**指向该类型**的**新分配**的**零值**的**指针**，主要为**值类型**分配内存
例：
```go
num1 := 100
fmt.Printf("num1的类型%T，num1的值%v，num1的地址%v"，num1，num1，&num1)
//num1的类型int，num1的值100，num1的地址0xcXXXXXXXX

num2 := new(int)
fmt.Printf("num2的类型%T，num2的值%v，num2的地址%v，num2指针指向的值"，num2，num2，&num2，*num2)
//num2的类型*int，
//num2的值0xcXXXXXXXX，num2的地址0xcXXXXXXXX，num2指针指向的值0,（这两个地址在程序运行时，系统动态分配）
//num2的值是一个地址，该地址指向int类型的零值0
```
![func12](./func12.jpg)

3、make：分配内存，主要用来分配**引用类型**