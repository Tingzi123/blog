---
title: 'go:错误处理机制、数组、切片、map'
date: 2020-06-11 15:29:33
tags:
---
### 1、go错误处理机制
1、Go引入处理方式：**defer、panic、recover**，通常三者结合使用
2、这几个异常使用场景简单描述：Go中可以抛出一个panic异常，然后在defer中通过recover捕获这个异常，然后正常处理
示例：
```go
func test(){
    num1 := 10
    num2 := 0
    res := num1/num2  //会报错，10/0错误，0不可以作为除数
    fmt.Println("res=",res)
}

func main(){
     test()//test函数报错，下面的代码不会执行
     fmt.Println("main下面的代码")
}
```
异常处理：
```go
func test(){
    //使用defer+recover结合捕获并处理异常
    defer func(){
        err :=recover()//recover()是一个内置函数，可以捕获异常
        if err != nil{//如果捕获到异常
             fmt.Println("err=",err)
             //这里可以把错误信息发送给管理员
        }
    }()//这里调用defer后的匿名函数
    num1 := 10
    num2 := 0
    res := num1/num2  //会报错，10/0错误，0不可以作为除数
    fmt.Println("res=",res)
}

func main(){
    test()//defer+recover结合捕获并处理异常后，使得下面的代码可以执行
     fmt.Println("main下面的代码")
}
```
##### 自定义错误
1、Go支持自定义错误，使用**errors.New**和**panic**内置函数
1）errors.New（“错误说明”），返回一个**error**类型的值，表示一个错误
2）panic内置函数，接收一个空接口interface{}类型的值（即任何值）作为参数。可以接收error类型的变量，输出错误信息，并退出程序。
例：
```go
//函数读取配置文件init.conf的信息
//如果文件名传入不正确，返回一个自定义错误
func readConf(name string) (err error){
     if name == "init.conf"{//如果捕获到异常
             //读取...
             return nil
     }else {
             //返回一个自定义错误
             return errors.New("读取文件错误")
     }
}

//测试
func test(){
    err := readConf(init.conf)
    if err != nil{
             //如果发生错误，输出错误，并终止程序
             panic(err)
        }
     fmt.Println("test后面的代码")
}

func main(){
     test()//test函数如果发生读取文件错误，panic会终止程序，下面的代码不会执行
     fmt.Println("main下面的代码")
}
```
### 2、数组
1、数组可以存放多个同一类型数据。数组也是一种数据类型，Go中，数组是**值类型**
2、四种初始化数组方式
如果定义时不赋值，则会被系统赋默认值
* 1）
```go
var arr [3]int = [3]int{1,2,3}
```

* 2）var arr时不指定类型，类型推导
```go
var arr = [3]int{1,2,3}
```

* 3）用“...”代替数组大小
```go
var arr = [...]int{1,2,3}
```

* 4）指定元素值的下标
```go
var arr = [3]string{1:"tom",2:"jack",0:"marry"}//顺序可乱序，输出按下标输出
```
##### 3、数组遍历
1、常规for循环遍历
```go
for i := 0; i < len(arr);i++{
    fmt.Printf("arr[%d]=%v\n]",i,arr[i])
}
```

2、for-range结构遍历
```go
for index,value := range arr{
    fmt.Printf("i=%v,v=%v\n",index,value)
}
```
##### 4、数组使用细节
1、var arr []int，这时arr是一个slice切片
2、Go数组属于值类型，默认情况下是值传递，进行值拷贝，数组间不会互相影响
3、如想在其他函数中修改原来的数组，可以使用引用传递（指针方式，传数组地址）
4、**长度是数组类型的一部分**，在传递函数参数时，需要考虑数组长度
```go
func modify(arr []int){
    arr[0]=100
     fmt.Println("modify的arr",arr)
}

func main(){
    var arr =[...]int{1,2,3}
    modify(arr)//这里编译错误，因为认为main函数中的[3]int与modify函数中的形参[]int不是同一类型
}
```
### 3、切片
1、切片是数组的一个引用，因此数组时引用类型，传递方式是引用传递
2、切片的长度可以变化，因此切片是一个动态数组
3、切片的使用和数组类似，遍历、访问切片的元素和求切片长度len（slice）都一样
3、切片定义的基本语法
```go
var slice []int//与数组不同的是，【】中不需要填入大小或“...”
```
4、示例
```go
//定义一个数组
var intArr [5]int = [...]int{1,22,33,66,99}

// intArr[1:3]表示切片slice从intArr这个数组的下标为1的元素开始引用，到下标为3的元素，但不包含标为3的元素
slice := intArr[1:3]
fmt.Println("slice的元素："，slice)              //[22,33]
fmt.Println("slice的元素个数："，len(slice))   //2
fmt.Println("slice的容量："，cap(slice))        //4

//切片的容量可以动态变化
```
##### 1、切片在内存中的布局
![slice1](./slice1.jpg)

由上图可以看出
1、slice是一个引用类型
2、slice从底层来说是一个结构体是struct，由三部分构成
```go
type slice struct{
    ptr *[2]intArr  //指向切片第一个元素的地址
    len int         //长度
    cap int         //容量
}
```
由于slice是引用类型，所以通过slice去修改引用到的intArr数组中的值，intArr本身的值也会发生改变

##### 2、切片使用的三种方式
1、定义一个切片，然后让切片去引用一个已经创建好的数组，如上所示
```go
//定义一个数组
var intArr [5]int = [...]int{1,22,33,66,99}

// intArr[1:3]表示切片slice从intArr这个数组的下标为1的元素开始引用，到下标为3的元素，但不包含标为3的元素
slice := intArr[1:3]
fmt.Println("slice的元素："，slice)              //[22,33]
fmt.Println("slice的元素个数："，len(slice))   //2
fmt.Println("slice的容量："，cap(slice))        //4

//切片的容量可以动态变化

//几种引用数组元素写法
slice1 := intArr[:3] //等价于intArr[0:3]  1,22,33
slice2 := intArr[1:] //等价于intArr[1:len(intArr)]  22,33,66,99
slice1 := intArr[:] //等价于intArr[0:len(intArr)]  1,22,33,66,99
```

2、通过make来创建切片
基本语法
```go
var slice []type = make([]type,len,cap)
//cap可选参数，cap>=len
```
使用案例
```go
var slice []float64
//对于切片，必须make后使用
fmt.Println("slice="，slice)//输出[]

//make后
slice = make([]float64,5,10)
fmt.Println("slice="，slice)//输出[0,0,0,0,0],默认值为0
```
3、定义切片时直接指定具体数组
```go
var slice []int = []int{1,3,5}
fmt.Println(slice)
```
注意：

* 方式1创建切片的方式是直接引用一个事先定义好的数组，程序员对这个数组可见
* 方式2通过make方式创建切片，**make也会创建一个数组，由切片在底层进行维护，程序员不可见**
* **切片定义完之后，还不能直接使用**，这时是一个空切片【】，需要引用到一个数组或者make一个空间来使用
* 切片可以再切片

##### 3、切片的遍历
1、for循环
```go
var arr [5]int = [...]int{10,20,30,40,50}
slice := arr[1:4] //20,20,30

for i := 0;i<len(slice);i++{
    fmt.Printf("slice[%v]=%v ",i,slice[i])
}
```
2、for-range
```go
var arr [5]int = [...]int{10,20,30,40,50}
slice := arr[1:4] //20,20,30

for i,v := range slice{
    fmt.Printf("i=%v v=%v\n",i,v)
}
```

##### 4、切片的使用注意事项
1、**切片定义完之后，还不能直接使用**，这时是一个空切片【】，需要引用到一个数组或者make一个空间来使用

2、切片可以再切片

3、用**append**内置函数，可以对切片动态追加
```go
    var slice []int = []int{100,200,300}
    //通过append直接给切片slice追加具体的值，数据类型需匹配
    slice = append(slice,400)
    //append操作后，会生成一个新数组，需要将append后的值赋给slice，保证slice引用到append操作后的新数组
    slice = append(slice,500)
    slice = append(slice,600,700,800)//一次追加多个
    slice = append(slice,slice...)//直接追加一个切片
    fmt.Println(slice)
```

* append操作的本质就是对数组扩容
* go底层会创建一个新的数组newArr，将slice原来包含的元素拷贝到新数组newArr中，slice再重新引用数组newArr
* 数组newArr在底层维护，程序员不可见

4、用**copy**内置函数，可以对切片拷贝
```go
    var a []int = []int{1,2,3,4,5}
   
    var slice = make([]int,10)
    fmt.Println(slice)//输出[0,0,0,0,0,0,0,0,0,0]
    
    copy(slice,a)//将切片a的值拷贝到slice中
    fmt.Println(slice)//输出【1，2，3，4，5，0，0，0，0，0】
```

* copy操作需要两个参数都是切片，只能从切片拷贝到切片
* 上面代码中，切片a与slice的数据空间是独立的，互不影响
* 将切片a拷贝到slice，slice的长度小于a也是正确的
* 切片是引用类型，在传递时遵守引用传递机制

##### 5、slice与string
1、string底层是一个byte数组，因此string也可以进行切片处理操作
```go
    str := "hello world"
    //使用切片获取到world
    slice := str[6:]
    fmt.Println(slice)//输出world
```
2、string在内存中的形式，以串“abcd”为例
![slice2](./slice2.jpg)

*string底层由两部分组成，指向一个字节数组的指针*[4]byte和长度len
字节数组中真正存放串的内容

3、string是不可变的，即不能通过str[0]='z'的方式来修改字符(编译会报错)

4、如果需要修改字符串，可以先将string->[]byte（或者 []rune）->修改->重写转成string
```go
    str := "hello world"
   //把‘w’改成h
   //首先转成byte切片，可以处理英文和数字，中文用[]rune
   //因为[]byte按字节来处理，而一个汉字占3个字节，因此会出现乱码，[]rune按字符处理，兼容汉字
   slice := []byte(str)
   slice[6]='h'
   str = string(slice)
   fmt.Println(slice)//输出hello horld
``` 

### 4、映射map
1、map是key-value数据结构，又称为字段或者关联数组

2、声明基本语法
```go
    var myMap map[keyType]valueType
``` 

* key可以是很多种类型，如：bool，int系列、string、指针、channel，还可以是只包含前面几个类型的接口、结构体、数组，**通常为int、string**
* **key不可以是slice、map、function**，因为这几个不能用“==”判断（key通常需要判断key存不存在）

* value的类型通常是string、map、struct

* key不可重复，value可重复，**key**是**无序的**

* **声明不会分配内存**，初始化需要make分配内存后才能赋值和使用 
```go
var a map[string]string //声明，还未分配内存

a["no1"]="宋江"  //这里会panic恐慌，给一个空map赋值，会报错
fmt.Println(a)   
```

使用map前需要先make，给map分配空间，和数组不一样，数组声明后会默认初始化为零值
```go
var a map[string]string //声明，还未分配内存

//使用map前需要先make，给map分配空间，和数组不一样，数组声明后会默认初始化为零值
a=make(map[string]string,10)//分配10个空间，不写默认为1
a["no1"]="宋江"
a["no2"]="吴用"

fmt.Println(a)        //输出 map 【no1:宋江 no2:吴用】
```

#### 1、map的使用方式
##### 1、声明->make->赋值
```go
var a map[string]string //声明，还未分配内存，这时map==nil

//使用map前需要先make，给map分配空间，和数组不一样，数组声明后会默认初始化为零值
a=make(map[string]string,10)//分配10个空间，不写默认为1
a["no1"]="宋江"
a["no2"]="吴用"

fmt.Println(a)        //输出 map 【no1:宋江 no2:吴用】
```

##### 2、声明同时make->赋值
```go
    city := make(map[string]string)
    city["no1"]="北京"
    city["no2"]="上海"
    city["no3"]="武汉"

    fmt.Println(city)        //输出 map 【no1:北京 no2:上海 no3:武汉】
```

##### 3、声明时直接赋值
```go
    city := map[string]string{
        “no1”："北京"，
        “no2”："上海"，
        “no3”："武汉"
    }

    fmt.Println(city)        //输出 map 【no1:北京 no2:上海 no3:武汉】
```

#### 2、map的增删改查（crud）操作
##### 1、增加和更新
```go
map["key"] = value
//1、如果key之前不存在，就是增加操作
//2、如果key之前存在，就是修改操作，新值覆盖旧值
```

##### 2、删除
```go
delete(map,"key")
```
1、delete是一个内置函数，如果key存在就删除该key-value，如果不存在，不操作，也不会报错
2、如果map为nil也不操作，不报错
3、如果我们要删除map所有的key，只能遍历逐个删除，不能一次全删除。或者map = make(map[keyType]valueType)，make一个新的，让原来的成为垃圾，被GC回收

##### 3、查找
```go
    city := map[string]string{
        “no1”："北京"，
        “no2”："上海"，
        “no3”："武汉"
    }
    
    //查找
    val,ok := city["no1"]
    if ok {
        fmt.Println("有key:no1，值为：%v"，val)
        }else {
         fmt.Println("没有key:no1")
     }
```

#### 3、map的遍历
map遍历只能通过for-range，不能用普通for循环，因为map没有下标，key是无序的
```go
    city := map[string]string{
        “no1”："北京"，
        “no2”："上海"，
        “no3”："武汉"
    }
    
    //遍历
    for k,v := range city {
        fmt.Printf("k=%v v=%v \n"，k,v)
    }
```

#### 4、map的长度
len(map)，统计map中有几对key-value
```go
    city := map[string]string{
        “no1”："北京"，
        “no2”："上海"，
        “no3”："武汉"
    }
    
        fmt.Println("city 有"，len（city），“对key-value”)
```

#### 5、map切片
切片的数据类型如果是map，则成为**map切片**，这样**map的个数可以动态变化**，map切片是一种切片，切片的每个元素都是一个map.
```go
    //声明一个map切片
    var monsters []map[string]string
    
    //切片make后才能使用
    monsters = make([]map[string]string,2)//len=2
    
    if monsters[0] == nil{
        //切片的元素是map，map需要先make再使用
        monsters[0] = make(map[string]string,2)//map有两个key
        monsters[0]["name"] = "牛魔王"
        monsters[0]["age"] = “500”
    }
    
    if monsters[1] == nil{
        //切片的元素是map，map需要先make再使用
        monsters[1] = make(map[string]string,2)//map有两个key
        monsters[1]["name"] = "玉兔精"
        monsters[1]["age"] = “400”
    }
    
    //因为一开始定义切片长度为2，如果还要添加元素，用append函数动态增加
    //1、定义一个monster信息
    newMonster := map[string]string{
        "name" = "新妖怪：红孩儿",
        "age" = "200"
    }
    
    //2、添加到切片中
    monsters = append(monsters,newMonster)
    
     fmt.Println(monsters)
```

#### 6、map排序
1、GO中没有专门的map排序方法
2、map默认是无序的，也不是按照添加顺序存放，每次遍历的输出结果也不一样
3、对map排序，可以先将key排序，再根据key值遍历输出
```go
    map1 := make(map[int]int,10)
    map1[10]=130
    map1[1]=13
    map1[4]=56
    map1[8]=90
        
    fmt.Println(map1)//这里每次的输出结果顺序可能不一致
    
    //排序
    //1、先将map的key放到切片中
    var keys []int
    for k,v := range map1{
        keys = append(keys,k)
    }
    
    //2、对切片排序
    sort.Ints(keys)
    fmt.Println(keys)//key递增输出
    
    //3、遍历切片，根据key值遍历输出值
    for _,k  := range keys{
        fmt.Printf("map1[%v]=%v \n",k,map1[k])
    }
```
#### 7、map使用细节
1、map是引用类型，遵守引用传递机制。
2、map容量到达后，想再增加元素，会自动扩容，不会panic，slice会panic
3、map的value也经常使用struct类型
