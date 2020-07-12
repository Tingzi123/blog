---
title: hexo博客迁移
date: 2020-07-02 15:53:23
tags:
---
#### 1、安装hexo博客必要的软件
1、下载安装Git客户端
2、安装node js

#### 2、Github添加新电脑生成的密钥
打开git bash输入如下命令：
```go
ssh-keygen -t rsa -C "xxxxx@xxx.com"
```
邮箱为GitHub注册邮箱，输入命令后直接回车，生成密钥对。根据提示找到密钥对所在位置，将id_rsa.pub公钥内容复制粘贴到Github-settings-‘SSH and GPG keys’-‘SSH keys’中。

使用
```go
ssh -T git@github.com   测试公钥是否添加成功
```
#### 3、备份原文件
需要转移的文件有：
![hexo1](./hexo1.png)
由于配置文件和主题文件需要经常更改，采用github创建博客分支的方式进行备份。

**1、创建分支**
 克隆github上上生成的静态文件到hexo文件夹中
```go
git clone https://github.com/yourname/xxxx.github.io.git hexo
```
克隆后将除.git文件外其他所有文件删除。主要是为了得到版本管理文件夹.git（.git文件为隐藏文件，可直接将可见文件全部删除）。

**2、将备份的原文件复制到此文件夹**
若文件夹是从github克隆，则需要删除主题文件中的版本控制文件夹,以next主题为例：
```go
$ rm -rf thems/next/.git*
```
创建名为hexo的分支
```go
$ git checkout -b hexo
```
保存所有文件到暂存区
```go
$ git add --all
```
提交变更
```go
$ git commit -m "hexo-2"
```
提交变更时报错：
![hexo2](./hexo2.png)
根据提示配置。
推送分支到github,并用–set-upstream与origin创建关联，将hexo设置为默认分支
```go
$ git push --set-upstream origin hexo
```
#### 4、迁移
以后在其他电脑上写博客，直接将分支克隆下来。再使用npm install安装依赖。
```go
$ git clone -b hexo https://github.com/yourname/xxx.github.io.git
$ npm install
```
#### 5、发表文章
1、新建文章
```go
hexo n "xxx"
```
2、注意：需要使用git push把源文件推到分支上
```go
$ git add .
$ git commit -m "xxxx"
$ git push origin hexo
```
3、部署文章
```go
hexo d -g
```
参考：
1、https://fl4g.cn/2018/08/03/Hexo%E5%8D%9A%E5%AE%A2%E8%BF%81%E7%A7%BB%E5%88%B0%E5%85%B6%E4%BB%96%E7%94%B5%E8%84%91/

2、https://blog.csdn.net/white_idiot/article/details/80685990