---
layout: post
title: "如何在Mac下配置Github和Bitbucket的SSH"
date: 2013-12-25 03:18:52 +0800
comments: true
categories: 
---

##设置Github的用户名和邮箱

```
git config --global user.name "你在Github上的昵称"
git config --global user.email "你在Github上的邮箱"
```
##生成SSH密钥过程
###查看是否已经有了SSH密钥

```
cd ~/.ssh
```
如果没有密钥则不会有此文件夹，有则备份删除

```
cp -R ~/.ssh ~/.ssh_bak
rm -R ~/.ssh  
```
###生成密钥

```
ssh-keygen -t rsa -C "你在Github上的邮箱"
```
第一次要输入file名字，直接回车即可，默认文件名为`id_`前缀，然后会被要求输入个密码并且确认。

###添加密钥到SSH

```
cd ~/.ssh
ssh-add -K id_rsa
```
需要刚才生成密钥时候输入的密码。

###在github上添加SSH Key

```
more id_rsa.pub
```
* 查看公钥里的内容并且全部复制下来(包括开头ssh-rsa和结尾邮箱)。
* 进入到`github`的`setting`，找到SSH Keys页面添加一个key，title随意，赋值公钥进去保存即可。

###测试是否成功

```
ssh git@github.com
```
成功则返回类似:

```
Hi molon! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```
###Bitbucket
[Bitbucket.org](http://bicbucket.org)是一个和`Github`极其类似的托管库网站，但是他的私有库是免费的，所以一般我们会把私有库放到`bitbucket`上，而其他库放到`github`上，毕竟`github`的`SNS`属性稍微多些，开源的库就尽量的希望有与其他人的更多互动。  
扯这些的目的呢，其实就是顺带说下，给`bitbucket`添加SSH Key实际上是一样的，我们只需要同样把`id_rsa.pub`里的内容复制，在`bitbucket`的`setting`页面添加即可。  
测试bitbucket ssh是否可连接

```
ssh -T git@bitbucket.org
```
成功则返回类似:

```
logged in as molon.

You can use git or hg to connect to Bitbucket. Shell access is disabled.
```
