---
layout: post
title: "IOS推送的实现-远程通知"
date: 2013-12-27 05:46:58 +0800
comments: true
categories: 
---
远程通知(即推送)的出现应该是由于苹果为了让APP后台时候一般不占用太多资源而创造的机制，像一般的IM系统，后台时候的聊天信息提醒就完全靠推送，实际上APP这时候应该是不做任何事情的。   
推送的机制：   

* APP去向系统注册通知
* 系统接受到了之后会为这个硬件上的APP向APNS(即苹果的推送服务器)请求一个唯一标识符，即为device token，用来标识当前苹果网络上，这个硬件上的这个APP的唯一身份。
* APNS把这个token传回给APP。
* APP把token传给其自己的服务端，服务端记录这个token在数据库，一般会对其进行和用户绑定。
* 若需要推送信息到APP，则服务端向APNS请求推送某信息到某一token的硬件上。
* APNS推送信息到对应硬件。

#引用
[Apple Push Notification Services in iOS 6 Tutorial: Part 1/2](http://www.raywenderlich.com/32960/apple-push-notification-services-in-ios-6-tutorial-part-1)   
[Apple Push Notification Services in iOS 6 Tutorial: Part 2/2](http://www.raywenderlich.com/32963/apple-push-notification-services-in-ios-6-tutorial-part-2)

#配置流程

##推送证书的配置和安装
参见笔者的另外一篇文章，[IOS开发者证书的配置和安装](/2013/12/ioskai-fa-zhe-zheng-shu-de-pei-zhi-he-an-zhuang/)   

##推送服务端的配置
###创建pem文件   
* 打开命令行，cd进入存放有cer和p12的文件夹。
* 输入以下命令   

```	
openssl x509 -in aps_development.cer -inform der -out cer.pem 
openssl pkcs12 -nocerts -out p12.pem -in MolonDevNotification.p12
cat cer.pem p12.pem > ck.pem

```
其中`aps_development.cer`和`MolonDevNotification.p12`根据实际名称来定。   
而第二个命令会让确认p12的导出密码，以及生成的pem对应使用密码。   

* 测试是否可用。

```
openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert cer.pem -key p12.pem 
#然后输入刚才设置的pem的密码即可
```
若返回的是类似下图即表示可用   
![测试结果](/img/20131227/QQ20131227-40.png)   
 
###编写PHP请求推送代码

```php

<?php  
  
// Xcode控制台输出的Device Token
//这个玩意是直接要全都是十六进制的那种字符串，没有空格，没有<>符号
$deviceToken = '要接受消息的硬件的token';  
  
// 生成p12.pem时候设置的pem使用密码  
$passphrase = 'pem密码';  
  
// 要推送的信息
$message = '你那么吊，你父母知道吗!';  
  
////////////////////////////////////////////////////////////////////////////////  
  
$ctx = stream_context_create();  
stream_context_set_option($ctx, 'ssl', 'local_cert', 'ck.pem');  
stream_context_set_option($ctx, 'ssl', 'passphrase', $passphrase);  
  
// 和APNS建立一个连接，这里因为是开发者的，所以是sandbox下
// 正式版连接的服务器是ssl://gateway.push.apple.com:2195
$fp = stream_socket_client(  
    'ssl://gateway.sandbox.push.apple.com:2195', $err,  
    $errstr, 60, STREAM_CLIENT_CONNECT|STREAM_CLIENT_PERSISTENT, $ctx);  
  
if (!$fp)  
    exit("Failed to connect: $err $errstr" . PHP_EOL);  
  
echo 'Connected to APNS' . PHP_EOL;  
  
// Create the payload body  
$body['aps'] = array(  
    'alert' => $message,  
    'sound' => 'default',
    'badge' => 1,
    );  
  
// Encode the payload as JSON  
$payload = json_encode($body);  
  
// Build the binary notification  
$msg = chr(0) . pack('n', 32) . pack('H*', $deviceToken) . pack('n', strlen($payload)) . $payload;  
  
// Send it to the server  
$result = fwrite($fp, $msg, strlen($msg));  
  
if (!$result)  
    echo 'Message not delivered' . PHP_EOL;  
else  
    echo 'Message successfully delivered' . PHP_EOL;  
  
// Close the connection to the server  
fclose($fp);  

?>

```

##编写App的代码
###注册通知
在`AppDelegate.m`里
```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	// 注册通知，把三种类型都注册了，用户允许的话，三个开关都会默认打开。
	[[UIApplication sharedApplication] registerForRemoteNotificationTypes:
		(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeAlert)];
 
    return YES;
}
```

###接收devide token
```objc
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    NSString *token = deviceToken.description;
    token = [token stringByReplacingOccurrencesOfString:@" " withString:@""];
    token = [token substringWithRange:NSMakeRange(1, token.length-2)];
    NSLog(@"token:%@",token); //这里的token就可以直接放到PHP的push文件里了。
}

- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
    NSLog(@"get token failed.");
}
```

###接收推送消息

```objc
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
{
    NSLog(@"%@",userInfo);
}
```

##推送消息的结构
```
{
    aps =     {
        alert = "你那么吊，你父母知道吗!";
        badge = 1;
        sound = default;
    },
    info = {
    }
}
```
这里是didReceiveRemoteNotification接收到的userInfo的结构，aps是系统用到的，必须要有，里面三个key分别对应`UIRemoteNotificationType`的`Alert`、`Badge`、`Sound`三种类型。而info是自定义添加的，想传啥传啥，但是也不是完全没限制，看下文php代码(结合上文完整php代码)

```php
// Encode the payload as JSON  
$payload = json_encode($body);  
  
// Build the binary notification  
$msg = chr(0) . pack('n', 32) . pack('H*', $deviceToken) . pack('n', strlen($payload)) . $payload;  
```
注意里面的`$payload`，他的长度不能大于256，如果大于，推送将不成功。

##自定义推送的声音
在工程里添加声音文件，例如a.caf，b.mp3。然后推送消息体里"sound" ＝> "a.caf"，即可。

#深入测试   
   * 硬件里`设置->通知中心->应用名称`中`提醒样式、应用程序图标标记、声音`三个选项分别对应`UIRemoteNotificationType`的`Alert`、`Badge`、`Sound`三种类型，修改他们则对应的`enabledRemoteNotificationTypes`都会引起变化。若三者都被关闭，则变成`None`。
   * APP在安装到手机上，第一次调用`registerForRemoteNotificationTypes`方法的时候会询问用户是否允许接受其通知。如果用户允许，`enabledRemoteNotificationTypes`则为刚才注册的几个通知类型*相与*，否则会是`None`。拒绝通知其实并不表示APNs不会Push过来，只是不显示没动静罢了。
   * 当`None`时不管联网与否，`didRegisterForRemoteNotificationsWithDeviceToken`方法不会执行，而非`None`时，联网状态下会执行，但未联网状态下若以前曾执行过一次，也会执行并获取上一次返回的token。
   * 若第一次就注册通知时候就没联网，`didRegister...DeviceToken`和 `didFail...WithError`方法都不会执行。
   * 用户拒绝了通知，只能在适当的时候检测并且提示其手动去改正，无法再像第一次那样请求是否允许。
   * 用户未联网时候，服务端也可以成功发送请求到APNs，等待用户联网之后，APNs会即刻把消息Push进来，所以每个消息最好有代表发送时间的参数，以方便应用确认时间。
   * 推送可自定义的声音可以是mp3格式，时间可以挺长，并且多个推送过来的话，偶尔会重叠播放，只是偶尔。。
   * 应用删除之后不会接受到对应推送消息，但再次安装之后，还是会把没接的接过来，和没联网一个样。
   * 应用开启状态下，即使状态栏在拉下状态遮挡应用界面时候，也不会外部提醒，并且通知列表不会添加任何项。
   * 应用最小化和锁屏状态下，有消息进来会外部提醒，并且保存在状态栏下拉通知列表内。即使点击一项进入应用，此项也不会删除。
   * 服务端发送消息中如果一直不包含badge或者其为0，应用就再也无法控制顶部通知列表项的删除，当然如果不想着删除，就要保证程序内的同一条Push消息的重用后逻辑。
   * Push消息中包含大于0的badge，然后在`didReceiveRemoteNotification`方法里  `setApplicationIconBadgeNumber:0`即可让顶部通知列表中关于本应用的所有消息删除，很遗憾要删只能删除全部的。针对于必须设置为0才能删除其的问题，而这个方式很容易影响到应用内其他地方关于badge的逻辑，一般是在`setApplicationIconBadgeNumber:0`之后再重新判断逻辑set回来。


