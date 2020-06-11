---
title: 'go:文件操作、json'
date: 2020-06-11 15:54:54
tags:
---
### 1、文件操作
#### 1、基本认识
1、文件在程序中是以 **流** 的形式来操作的

* 流：数据在数据源（文件）和程序（内存）之间经历的路径
* 输入流：数据从数据源（文件）到程序（内存）之间的路径
* 输出流：数据从程序（内存）到数据源（文件）之间的路径

![file1](file1.png)

2、GO中，os.File封装所有文件相关操作（方法），File是一个结构体

3、**文件是引用类型**

#### 2、文件操作
##### 1、打开文件
```go
func Open(path string) (file *File,err error)
```
Open打开一个文件用于读取，如果操作成功，返回一个文件对象（结构体指针或者文件句柄）可用于读取数据；如果出错，错误底层类型是PathError

##### 2、关闭文件
```go
func (f *File ) Close (error) error
```
Close关闭文件f，用于读取，使文件不能用于读写。返回可能出现的错误

案例：
```go
func main(){
    //打开文件
    file , err := os.Open("d:/test.txt")
    if err != nil {
        fmt.Println("open file err=",err)
    }
    
    //输出文件
    fmt.Printf("file=%v",file)//输出 file=&<0xcXXXXXXXX>，
    //看出文件是一个地址（指针）
    
    //关闭文件
    err = file.Close()
    if err != nil {
        fmt.Println("close file err=",err)
    }
}
```

##### 3、读文件操作应用实例
1、读取文件的内容并显示在终端（**带缓冲区**的方式）
os.Open()、 file.Close()、bufio.NewReader()
```go
func main(){
    //打开文件
    file , err := os.Open("d:/test.txt")
    if err != nil {
        fmt.Println("open file err=",err)
    }
    
    //当函数退出时，要及时关闭文件
    defer  file.Close() //及时关闭file句柄，防止内存泄漏
    
    //创建一个 *Reader，是带缓冲的
    //默认缓冲4096个字节
    reader := bufio.NewReader(file)
    
    //循环读取文件的内容
    for{
        str ,err :=  reader.ReadString("\n")//读到一个换行就结束一次
        if err == io.EOF {//io.EOF表示文件的末尾
            break
         }
         
         //输出内容
         fmt.Print(str)
    }
    //文件读取结束
     fmt.Println("文件读取结束")
}
```

2、读取文件的内容并显示在终端（使用ioutil一次性将整个文件读入到内存中），这种方式适合文件不大的情况 。

* 这种方式不用显示的Open和Close文件，因为文件的Open和Close被封装到ReadFile函数内部

```go
func main(){
    //打开文件
    file = "d:/test.txt"
    
    //使用ioutil.ReadFile一次性将文件读取完
    content,err := ioutil.ReadFile(file)//返回一个[]byte（byt切片）
     if err != nil {
        fmt.Println("read file err=",err)
    }
    
    //把读取的文件内容显示到终端
    fmt.Printf("content=%v",content)//这是一个[]byte
    
    fmt.Printf("content=%v",string(content))//[]byte转成string输出
       
}
```
##### 4、写文件操作应用实例
1、基本介绍
```go
func OpenFile(path string,flag int,perm FileMode) (file *File,err error)
```

* path：要打开文件的路径
* flag：打开文件的方式
* perm：文件权限，Windows系统下无效，Unix和Linux下有效

###### 1、实例一
 带缓冲bufio.NewWriter
 
1）创建一个**新文件**，**写入**内容
```go
func main(){
    filePath := "d:\abc.test"
    //打开文件
    file , err := os.OpenFile(filePath,os,O_WRONLY | os.O_CREATE,0666)
    //以只写方式打开，如果没有文件就创建一个，如果有就继续往下
    
    if err != nil {
        fmt.Println("open file err=",err)
        return
    }
    
    //当main函数退出之前，要及时关闭文件
    defer  file.Close() //及时关闭file句柄，防止内存泄漏
    
    
    //写入内容
    str := "要写入的内容\r\n"
    //有的编辑器\n会换行
    //有的编辑器\r才会换行（如记事本）
    
    //写入时，使用带缓冲的*Writer
    writer := bufio.NewWriter(file)
    
    //循环写入内容
    for i:=0;i<5;i++{//写5次
        writer.writerString("str")
    }
    
    //因为Writer是带缓存的，
    //因此在调用writerString方法时，内容是先写入缓存的，
    //所以需要调用Flush方法，将缓冲的数据真正写入到文件中，
    //否则文件中会没有数据
    writer.Flush()
}
```

2）打开一个**已经存在**的文件，将原来的内容**覆盖**成新的内容
```go
func main(){
    filePath := "d:\abc.test"
    //打开文件
    file , err := os.OpenFile(filePath,os,O_WRONLY | os.O_TRUNC,0666)
    //以只写方式打开，如果文件已经有内容就清空
    
    if err != nil {
        fmt.Println("open file err=",err)
        return
    }
    
    //当main函数退出之前，要及时关闭文件
    defer  file.Close() //及时关闭file句柄，防止内存泄漏
    
    
    //写入要覆盖的新内容
    str := "写入要覆盖的新内容\r\n"
    //有的编辑器\n会换行
    //有的编辑器\r才会换行（如记事本）
    
    //写入时，使用带缓冲的*Writer
    writer := bufio.NewWriter(file)
    
    //循环写入内容
    for i:=0;i<10;i++{//写5次
        writer.writerString("str")
    }
    
    //因为Writer是带缓存的，
    //因此在调用writerString方法时，内容是先写入缓存的，
    //所以需要调用Flush方法，将缓冲的数据真正写入到文件中，
    //否则文件中会没有数据
    writer.Flush()
}
```

3）打开一个**已经存在**的文件，在原来的内容**追加**新的内容
```go
func main(){
    filePath := "d:\abc.test"
    //打开文件
    file , err := os.OpenFile(filePath,os,O_WRONLY | os.O_APPEND,0666)
    //以只写方式打开，在文件末尾以追加的形式写入内容
    
    if err != nil {
        fmt.Println("open file err=",err)
        return
    }
    
    //当main函数退出之前，要及时关闭文件
    defer  file.Close() //及时关闭file句柄，防止内存泄漏
    
    
    //写入要追加写入的新内容
    str := "追加写入的新内容\r\n"
    //有的编辑器\n会换行
    //有的编辑器\r才会换行（如记事本）
    
    //写入时，使用带缓冲的*Writer
    writer := bufio.NewWriter(file)
    
    //循环写入内容
    for i:=0;i<10;i++{//写5次
        writer.writerString("str")
    }
    
    //因为Writer是带缓存的，
    //因此在调用writerString方法时，内容是先写入缓存的，
    //所以需要调用Flush方法，将缓冲的数据真正写入到文件中，
    //否则文件中会没有数据
    writer.Flush()
}
```

4）打开一个已经存在的文件，将原来的内容**读出**显示在终端，并**追加**新的内容
```go
func main(){
    filePath := "d:\abc.test"
    //打开文件
    file , err := os.OpenFile(filePath,os,O_RDWR | os.O_APPEND,0666)
    //以读写方式打开，在文件末尾以追加的形式写入内容
    
    if err != nil {
        fmt.Println("open file err=",err)
        return
    }
    
    //当main函数退出之前，要及时关闭文件
    defer  file.Close() //及时关闭file句柄，防止内存泄漏
    
    //先读取文件原来的内容，并显示在终端
     //创建一个 *Reader，是带缓冲的
    //默认缓冲4096个字节
    reader := bufio.NewReader(file)
    
    //循环读取文件的内容
    for{
        str ,err :=  reader.ReadString("\n")//读到一个换行就结束一次
        if err == io.EOF {//io.EOF表示文件的末尾
            break
         }
         
         //输出内容，显示在终端
         fmt.Print(str)
    }
    
    //写入要追加写入的新内容
    str := "第二次追加写入的新内容\r\n"
    //有的编辑器\n会换行
    //有的编辑器\r才会换行（如记事本）
    
    //写入时，使用带缓冲的*Writer
    writer := bufio.NewWriter(file)
    
    //循环写入内容
    for i:=0;i<5;i++{//写5次
        writer.writerString("str")
    }
    
    //因为Writer是带缓存的，
    //因此在调用writerString方法时，内容是先写入缓存的，
    //所以需要调用Flush方法，将缓冲的数据真正写入到文件中，
    //否则文件中会没有数据
    writer.Flush()
}
```

###### 2、实例二
将一个文件的内容，写入到另外一个文件。（两个文件已经存在）
ioutil.ReadFile/ioutil.WriteFile
```go
func main(){
    //将”d:/test.txt“文件内容导入到”e:/kkk.txt“
    
    //1、将”d:/test.txt“文件内容读取到内存
    //2、将读取到的内容写入”e:/kkk.txt“
    
      file1Path = "d:/test.txt"
      file2Path = "e:/kkk.txt"

    //使用ioutil.ReadFile一次性将文件读取完
    data,err := ioutil.ReadFile(file1Path)//返回一个[]byte（byt切片）
     if err != nil {
        fmt.Println("read file err=",err)
        return
    }
    
    //把读取的文件内容显写入e:/kkk.txt文件
    err = ioutil.WriteFile(file2Path,data,0666)//data接收一个[]byte
     if err != nil {
        fmt.Println("write file err=",err)
        return
    }
}
```

##### 5、判断文件或文件夹是否存在
```go
func Stat(filePath string) (fi FileInfo,err error)
```

* 如果返回的error为nil，说明文件或文件夹存在
* 如果返回的error使用为os.IsNotExist判断为true，说明文件或文件夹不存在
* 如果返回的error为其他类型，则不确定是否存在

### 2、json
1、JSON（JavaScript Object Notation）是一种轻量级的数据交换格式。易于人阅读和编写，也易于机器解析和生成，并有效提升网络效率。

2、json序列化（如struct、map、slice序列化成json字符串）
json.Marshal()序列化后是一个[]byte，要转string      
```go
package main
import (
    "encoding/json"
    "fmt"
)

type Monster struct {
    Name string
    Age int    
    Birthday  string    
    Sal float64    
    Skill string
}

func testStruct(){
        monster := Monster{
        Name:"牛魔王",            
        Age:500,            
        Birthday:"2011-11-11",           
        Sal:8000.0,            
        Skill:"牛魔拳",      
    }      

//将monster序列化     
data,err:=json.Marshal(&monster)      
if err!=nil{            
    fmt.Printf("序列化失败 err=%v\n",err)     
}     

    //输出序列化后的结果     
    //json.Marshal(&monster)序列化后是一个[]byte，要转string      
    fmt.Printf("序列化后的结果monster=%v",string(data))
}

func main() {
    testStruct()
}
```
输出：
```go
序列化后的结果monster={"Name":"牛魔王","Age":500,"Birthday":"2011-11-11","Sal":8000,"Skill":"牛魔拳"}
```
注意：也可以对基本数据类型进行序列化，序列化结果直接将基本数据类型转成string，但是对基本数据类型进行序列化一般没有意义。

3、json反序列化（如将json反序列化成struct、map、slice）
json.Unmarshal([]byte(str),&monster)第一个参数要传一个[]byte   
```go
package main
import (
    "encoding/json" 
    "fmt"
)
//反序列化

type Monster struct {
    Name string    
    Age int    
    Birthday  string     
    Sal float64    
    Skill string
 }

func unSerialStruct(){  
    str:="{\"Name\":\"牛魔王\",\"Age\":500,\"Birthday\":\"2011-11-11\"," +            "\"Sal\":8000,\"Skill\":\"牛魔拳\"}"     

    //定义一个Monster实例      
    var monster Monster      

    //json.Unmarshal([]byte(str),&monster)第一个参数要传一个[]byte     
    err := json.Unmarshal([]byte(str),&monster)
    //传地址才能改变var monster的值
      if err!=nil{ 
         fmt.Printf("反序列化失败 err=%v\n",err)    
      }      
      
      fmt.Printf("反序列化后的结果monster=%v",monster)
 }
      
 func main() {   
        unSerialStruct()  
 }

```
输出：
```go
反序列化后的结果monster={牛魔王 500 2011-11-11 8000 牛魔拳}
```