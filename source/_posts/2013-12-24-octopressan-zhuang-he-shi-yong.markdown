---
layout: post
title: "Octopress安装和使用"
date: 2013-12-24 16:28:41 +0800
comments: true
categories: 
---
##前提
本文是以Mac安装为前提的，其他系统请自行选取合适自己的尝试。
##安装组件
###安装rvm
注意安装RVM应该是需要MacPort，而MacPort一般需要XCode的CommandLineTools组件，所以需要自行解决这两个东西先。

```
sed -i .bak 's!ftp.ruby-lang.org/pub/ruby!ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db
curl -L https://get.rvm.io | bash -s stable --ruby
```
* 第一个是修改rvm的下载源，否则国内访问这些资源超级慢的。
* 第二个是下载安装rvm，然后会自动下载安装ruby最新版，然后会自动下载安装rubygems最新版(根据命令行返回结果显示，如果这些都做了的话，下一步可以省略。)

###安装ruby和rubygems
```
rvm install 1.9.3 #只是最少1.9.3，以上版本亦可
rvm use 1.9.3
rvm rubygems latest
ruby --version
```
如果以后遇到了一些问题，首先就要确认是不是`ruby`和`rake`(在下面**生成和预览**下有提到)的版本问题。`ruby --version`用来检查版本最低是1.9.3，如果不是则使用`rvm use 1.9.3`切换当前有效ruby，不建议卸载系统自带的版本，可能会引起其他诸多问题。
###配置rubygems
```
gem sources -r https://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l
```
`gem sources -l`用来确认gem的资源URL是否只有[http://ruby.taobao.org](http://ruby.taobao.org)，也是一个国内的资源镜像。

##安装Octopress
###从Git获取源码
```
git clone git://github.com/imathis/octopress.git octopress
cd octopress
```
###安装Bundler
```
gem install bundler
bundle install
```
###安装Octopress默认主题
```
rake install
```
###生成和预览
```
rake generate
rake preview
```
预览时在命令行Ctrl+C退出预览。   
**第一次应该不会，但是以后如果遇到了显示rake版本不对的问题。可能是因为有两个rake版本存在，可以使用`gem unstall rake`，然后选择一个不需要的版本卸载即可**
##简单使用
###安装其他主题
把主题文件夹下载下来，放在根目录，例如名字是叫XXX   
```
cp -R XXX .themes/XXX
rake "install[XXX]" #以前有主题则需要输入y/n来确认
rake generate
rake preview #开启预览
```
* 第一行命令是把源码复制到.themes隐藏文件夹里。   
* 其实只要开一个命令行`rake preview`，不停止。然后再开一个命令行做添加文章，修改样式还有界面，修改主题等等rake方面的操作是可以的。可以认为`rake preview`是打开了一个Web服务器，并且其可以监视一些文件改动并做处理。
* 至于安装主题模板其实就是把.themes里的一些东西赋值到根目录的sass和source文件夹里。一般主题安装完还是需要自定义修改一些东西(在根目录下的修改)，修改完了之后把sass和source文件夹里的东西备份下来(不包含_post或者其他自己添加的文件夹)，就可以作为模板备份，即使以后需要重装，安装备份的模板就可以。

###基本设置
* 基本的一些博客标题还有个人信息的设置在_config.yml里，**需要注意的是每个设置的`:`号后面需要有个空格**，否则会出问题。
* 设置的具体含义不做解释了，但是像里面的一些对第三方东西例如facebook，tweet等等国内访问有问题一般笔者都是直接设置为false的，否则会引起页面加载特别不流畅。   
* 若要自定义一些页面内容细节，在source文件夹里找到对应页面，`{{site.XXXXX}}`就标识_config.yml里XXXXX设置的内容，可以自定义一些设置。

###发布文章
* `rake new_post['文章标题']`此命令即可在source的_post文件夹下面添加一个markdown文件。
* 打开markdown文件添加内容保存即可在浏览器预览页面看到新文章信息，注意头部文章标题和时间等等信息，这是网站所需要的，不能删除。
* markdown编辑器:[Mou(非网页)](http://www.mouapp.com/)，[马克飞象](http://maxiang.info/)，[MaHua](http://mahua.jser.me/)

###添加页面
* `rake new_page['img']`此命令在source文件夹下添加一个名为img的文件夹。
* 一般笔者是添加一个名为img的文件夹，然后把所有的图片资源都放到里面。而markdown编辑下添加图片的地方用/img/XXX.jpg即可。

###发布到Github
* 在Github下建议个名为XXX.github.com的Repository。
* 运行`rake setup_github_pages`，它会提示你输入你的github的库地址，例如：`git@github.com:yourname/XXX.github.com.git`
* 运行`rake generate`生成静态页面，`rake deploy`部署到github上。
* 如果deploy失败，查看错误消息。
	* 没有访问github的权限，查看笔者的另外一篇文章[如何在Mac下配置Github的SSH](/2013/12/ru-he-zai-ben-ji-pei-zhi-githubde-ssh/)
	* 需要先pull下来，然后才能push，一般这种情况我都是在github上删了仓库重新建个空的，然后deploy。

其实部署到网络的只有public文件夹的所有内容(也就是`rake generate`命令生成的内容)，如果你有其他的空间，例如BAE，你也可以将public文件夹里的内容放到其上，同样可以访问，只要能运行html文件的地方，BAE的话需要设置app.conf里的主页为index.html。PS:`rake deploy`命令不可用于除了Github和Heroku外的其他空间。

###Github自定义域名
* 在source根目录下建议一个文件CNAME(无后缀),里面只写一行即为你的域名无前缀，例如molon.me
* 笔者是使用的[DNSPod](http://DNSPod.cn)作为域名解析服务器，对域名这块不甚了解，一张图解释设置。   
![github_dnspod_setting](/img/github_dnspod_setting.png)





















