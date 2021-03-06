---
title: GO：工作区和GOPATH
date: 2020-07-10 14:27:34
tags:
---
### 1、Go基础知识导图
![go1.jpg](./go1.jpg)

### 2、工作区和GOPATH
#### 1、Go 3个环境变量

 - GOROOT：Go语言安装根目录的路径，也就是GO语言的安装路径
 - GOPATH：若干工作区目录的路径，是我们自己定义的工作空间
 - GOBIN：Go程序生成的可执行文件的路径

#### 2、GOPATH有什么意义

 1. 可以把GOPATH简单理解成Go语言的工作目录，它的值是一个目录的路径，也可以是多个目录的路径，每个目录都代表Go语言的一个工作区。
 
 2. 我们需要利用这些工作区去放置Go语言的源码文件，以及安装（install）后的归档文件（archive file，也就是以.a为扩展名的文件）和可执行文件。
 
 3. Go语言项目在其生命周期内的所有操作（编码、依赖管理、构建、测试、安装等）基本都是围绕GOPATH和工作区进行的。

go build 命令一些可选项的用途和用法
#### 1、Go 语言源码的组织方式是怎样的？
1、Go语言的源码也是以代码包为基本组织单位的。在文件系统中，这些代码包与目录一一对应。目录有子目录，代码包也可以有子包。

2、一个代码包可以包含任意个以.go为扩展名的原码文件，这些源码文件都需要被声明属于同一个代码包。

3、代码包的名称一般会与源码文件所在目录同名。如果不同名，在构建、安装时会以代码包名称为准。

4、每个代码包都有导入路径。代码包的导入路径是其他代码在使用该包中的程序实体时，需要引入的路径。例如：
```go
import "github.com/labstack/echo"
```
5、在工作区中，一个代码包的导入路径实际上就是从src子目录到该包的实际存储位置的相对路径。

6、所以说，Go语言源码的组织方式就是以环境变量GOPATH、工作区、src目录个代码包为主线的。一般情况下，Go语言的源码文件都需要存放在环境变量GOPATH包含的某个工作区（目录）中的src目录下的某个代码包（目录）中。

#### 2、源码安装后的结果是什么？（只有在安装后，Go语言源码才能被我们或其他代码使用）
1、源码文件通常会被放在某个工作区的src子目录下

2、安装后如果产生了归档文件（以.a为扩展名的文件）会被放进该工作区的pkg子目录

3、如果产生了可执行文件，就可能会被放在该工作区的bin子目录下。

**归档文件存放具体位置和规则：**
1、源码文件会以代码包的形式组织起来，一个代码包其实就对应一个目录。安装某个代码包而产生的归档文件就是与这个代码包同名的。

2、放置它的相对目录就是该代码包的导入路径的直接父级。比如，一个已存在的代码包的导入路径是：
```go
github.com/labstack/echo
```
那么执行命令
```go
go insatll github.com/labstack/echo
```
生成的归档文件的相对目录就是github.com/labstack，文件名为echo.a。

3、上面这个代码包导入路径还有一层含义，即：该代码包的源码文件存在于github网站的labstack组的代码仓库echo中

4、归档文件的相对目录与pkg目录之间还有一级目录，称为平台相关目录。平台相关目录的名称是由“build”的目标操作系统、下划线和目标计算架构的代号组成的。比如，构建某个代码包的目标擦欧总系统时Linux，目标计算架构是64位，对应的平台相关目录就是linux_amd64。
因此，上述代码包的归档文件就会被放置在当前工作区的子目录
pkg/linux_amd64/github.com/labstack中。
gopath与工作区：
![gopath1.png](./gopath1.png)

5、某个工作区的src子目录下的源码文件在安装后一般会被放置到当前工作区的pkg子目录下对应的目录中，或者直接被放置到该工作区的bin子目录中。

#### 3、理解构建和安装Go程序的过程
构建与安装的异同：
1、构建使用命令go build，安装使用命令go install。构建和安装代码包的时候都会执行编译、打包等操作，并且，这些操作生成的任何文件都会先被保存到某个临时的目录中。

2、构建源码文件

 - 如果构建的是库源码文件

那么操作后产生的结果只会存在于临时目录中，这里构建的主要意义在于检查和验证；

 - 如果构建的是命令源码文件

操作的结果文件会被搬运到源码文件所在的目录中。

3、安装过程会先执行构建、然后还会进行你链接操作，并且把结果文件搬运到指定目录。如：

 - 安装库源码文件，结果文件被搬运到他所在工作区的pkg目录下的某个子目录中；
 - 安装命令源码文件，结果文件被搬运到他所在工作区的bin目录中，或者环境变量GOBIN指向的目录中。

### 4、go build 命令一些可选项的用途和用法
1、在运行go build命令的时候，默认不会编译目标代码包所依赖的那些代码包。

2、如果被依赖的代码包的归档文件不存在，或者源码文件有了变化，那它还是会被编译。

3、如果要强制编译它们，可以在执行命令的时候加入标记-a。此时，不但目标代码包总是会被编译，它依赖的代码包也总会被编译，即使依赖的是标准库中的代码包也是如此。

5、另外，如果不但要编译依赖的代码包，还要安装它们的归档文件，那么可以加入标记-i。

6、那么我们怎么确定哪些代码包被编译了呢？有两种方法。

 - 运行go build命令时加入标记-x，这样可以看到go build命令具体都执行了哪些操作。另外也可以加入标记-n，这样可以只查看具体操作而不执行它们。
 - 运行go build命令时加入标记-v，这样可以看到go build命令编译的代码包的名称。它在与-a标记搭配使用时很有用。

7、下面再说一说与 Go 源码的安装联系很紧密的一个命令：go get。
命令go get会自动从一些主流公用代码仓库（比如 GitHub）下载目标代码包，并把它们安装到环境变量GOPATH包含的第 1 工作区的相应目录中。如果存在环境变量GOBIN，那么仅包含命令源码文件的代码包会被安装到GOBIN指向的那个目录。

最常用的几个标记有下面几种。
```go
-u：下载并安装代码包，不论工作区中是否已存在它们。

-d：只下载代码包，不安装代码包。

-fix：在下载代码包后先运行一个用于根据当前 Go 语言版本修正代码的工具，然后再安装代码包。

-t：同时下载测试所需的代码包。

-insecure：允许通过非安全的网络协议下载和安装代码包。HTTP 就是这样的协议。
```