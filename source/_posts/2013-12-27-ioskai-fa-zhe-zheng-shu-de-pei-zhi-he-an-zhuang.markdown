---
layout: post
title: "IOS开发者证书的配置和安装"
date: 2013-12-27 04:04:39 +0800
comments: true
categories: 
---
#前提
读者若没有购买Apple开发者资格的话，还请先做好这一步。*本文只针对开发版来解释，一些理论和想法全凭笔者的猜测，并不一定能套用在发布版上，请自行斟酌。*

##设置和工具
* [https://developer.apple.com/account/ios/certificate/certificateList.action](https://developer.apple.com/account/ios/certificate/certificateList.action)苹果开发者证书管理页面。
* Finder -> 应用程序 -> 实用工具 -> 钥匙串访问
 
##一些理论
* Bundle Identifier，这个东西可以理解为是APP的唯一标识符，一般形式为com.companyName.appName。一部手机里一种标识符的APP只能存在一个，如果安装相同标识符的APP之前的会被覆盖。但是一般Apple会给每一位开发者分配一个唯一的标识符前缀(被称为Bundle Seed ID)，上架的APP实际的标识符一般形式为XXXX.com.companyName.appName。
* 不对称加密，公钥和私钥。简单来说，就是公钥加密的东西只有私钥才能解密，私钥加密的东西只有公钥才能解密，至于为什么，在下觉得这是数学家需要考虑的事，不必深究。而这样做的用途大概也可分为俩种：
	* A用私钥加密某文件发往其他处，有公钥的人解密之后获得文件就说明这文件必然是经了A手的，因为公钥能解密必定只有拥有私钥的A才有加密的资格，可以称之为电子签名。
	* B用公钥加密了某文件发给A，A用私钥解密获得文件。这样加密的文件就只有A能解密，因为只有A有私钥。
* Certificate(证书)，Certificate Authority(证书颁发机构)，Certificate Signing Request(CSR 证书签名请求文件)，Provisioning Profiles(配置概要文件，在开发版可以理解为appID，开发者证书，硬件三者绑定在一块的文件，而这个文件需要存在绑定的硬件里才能用其调试APP，算是苹果限制硬件调试用到的东西。)

##大概流程和猜测
开发者从Certificate Authority(证书颁发机构)申请一份CSR(证书签名请求文件)，申请成功之后会生成一个私钥在开发者机器上，将CSR上传到苹果那边，然后就会获得证书。
开发者在拥有私钥的机器上生成APP，即把APP通过私钥签名(即Code Signing)，并且指定了Provisioning Profile之后。用作调试的硬件会保存这个Profile，并且通过其中的证书存储的公钥解密验证开发者身份，以及其中的硬件ID验证自身是否具有调试资格。这样整个验证就打通了，即可真机调试。   
对于笔者而言，暂时了解那么多，已经足够了，若需要对其深入理解，请参看:[iOS Code Signing: 解惑](http://www.cnblogs.com/andyque/archive/2011/08/30/2159086.html)

#配置流程
这里以开发者推送证书为例，完全无脑傻瓜图片教程。按步操作绝对不会出错。

##安装证书颁发机构
其实我并不知道它到底有没有起作用，因为在申请CSR的时候并没有去指定选择它。

![苹果证书颁发机构(Worldwide Developer Relations Certificate Authority)](/img/20131227/QQ20131227-01.png)

如果钥匙串访问中未发现这个证书(即苹果证书颁发机构)，需要[下载(苹果官方链接)](https://developer.apple.com/certificationauthority/AppleWWDRCA.cer)安装。

##申请CSR和保存私钥
请记得阅读每张图片下面的标签。   

![申请CSR](/img/20131227/QQ20131227-02.png)   
![保存到硬件，后缀为.certSigningRequest](/img/20131227/QQ20131227-03.png)   
![钥匙串访问中可以看到私钥(专有密钥)和公钥](/img/20131227/QQ20131227-04.png)   
![导出私钥，后缀为.p12](/img/20131227/QQ20131227-05.png)   
![p12文件的密码，记录下以后要用到](/img/20131227/QQ20131227-06.png)   
![当前Mac登录管理员的密码](/img/20131227/QQ20131227-07.png)   

这里面获取到的一份.certSigningRequest和.p12文件一定要保存好，前者可以重复传递给苹果获取一份新的证书，但证书虽新，它依然是对应同一个私钥，即p12，至于为什么要获取新证书，最大的原因就是因为所有的证书都是会过期的。而p12更应该保存好，因为全世界只有你一个人有，没有它证明不了你的身份。

##添加AppID
![添加AppID](/img/20131227/QQ20131227-08.png)   
![设置名称以及其对应的Bundle ID](/img/20131227/QQ20131227-09.png)   
这里选用Explicit App ID是因为有推送，内购，Game Center等其他功能的证书必须是明确的Bundle ID，不能是使用了通配符`*`的，Wildcard App ID则相反，但是其只能作为一般调试作用。看得懂英文的话看这篇:[When should I use a wildcard App ID?](https://developer.apple.com/library/ios/qa/qa1713/_index.html)   
![选择此App ID所需要的功能，这里选择推送](/img/20131227/QQ20131227-10.png)   
这样就完成了App ID的添加。   

##生成证书
![添加证书](/img/20131227/QQ20131227-11.png)   
![选择开发者推送证书](/img/20131227/QQ20131227-12.png)   
![这里是苹果提醒我们注意证书颁发机构这个东西，下面是我在上文给的链接](/img/20131227/QQ20131227-13.png)   
![选择对应的AppID，推送证书只能选择有推送功能的AppID](/img/20131227/QQ20131227-14.png)   
![上传申请的CSR文件](/img/20131227/QQ20131227-15.png)   
![证书添加完毕，下载下来。后缀为.cer](/img/20131227/QQ20131227-16.png)  

##添加设备

![连接设备，找到其UDID](/img/20131227/QQ20131227-37.png)   
![添加设备，设置名称和UDID](/img/20131227/QQ20131227-38.png)   

##添加配置概要文件
 
![添加PP(姑且认为是简称)](/img/20131227/QQ20131227-17.png)   
![选择开发版](/img/20131227/QQ20131227-18.png)  
![选择App ID](/img/20131227/QQ20131227-19.png)  
![选择证书](/img/20131227/QQ20131227-20.png)  
![选择设备](/img/20131227/QQ20131227-21.png)  
![设置名称并且生成文件，下载下来，后缀为.mobileprovision](/img/20131227/QQ20131227-22.png)  

##具体用途
![最后获得的4个文件](/img/20131227/QQ20131227-23.png)  

* .p12和.certSigningRequest文件在上面已经说过了，保存下来会有用的。.p12在配置推送的服务端的时候也会用到，在其他文章里会说。
* .cer文件一般直接打开安装到本机机器。
* .mobileprovision文件也是直接打开，XCode会对其进行记录处理。   

下面是.cer和.mobileprovision的具体用处。   
![建立工程需AppID对应的Bundle ID](/img/20131227/QQ20131227-31.png)  
![Code Signing选择配置的证书，证书需要安装了才显示](/img/20131227/QQ20131227-32.png)  
![安装了.mobileprovision文件之后这里才可以选择](/img/20131227/QQ20131227-33.png)  
![一般只要调试运行过一次，手机里自然会保存这个PP](/img/20131227/QQ20131227-34.png)  
![如果没保存，有异常情况，这里可以直接拖过去](/img/20131227/QQ20131227-35.png)  


