title: 通过新浪SAE搭建自己的个人博客
id: 371
permalink: 371
categories:
  - PHP
date: 2012-04-23 01:02:55
tags:
---

新浪SAE是个好东西，对于学习PHP的同学是一个不错的练习空间，（现在已经推出了java了，不过需要邀请码）不用在四处寻找廉价的主机，而且 平台上已经集成了很多现成的应用，博客空间有wordpress，emlog等，PHP框架有thinkPHP,cakephp等，而且还是 ecshop，最土等电子商务系统，更方便的应用于用户，感兴趣的同学可以体验一下，现在我将写一篇关于搭建wordpress博客应用和自己创建一个新 浪微博的简单应用来抛砖引玉，让更多的同学朋友了解SAE。

SAE地址 [http://sae.sina.com.cn/](http://rrurl.cn/fzgC5N)

需要准备：新浪微博账号（必须），自己注册的域名（可选），SAE提供二级域名的访问，xxx.sinaapp.com 也可以绑定自己的域名，注意这里的域名可以不用备案哦~

# 搭建wordpress博客应用

具体步骤
<!-- more -->

## 用新浪微博账号授权
打开SAE地址 [http://sae.sina.com.cn/](http://rrurl.cn/fzgC5N) 点击“用新浪微博账号”登录，用新浪微博账号登录后点击导航条上的“应用仓库”，选择下面的wordpress

## 点击安装些应
点击安装些应，选择基于URL安装，并且在填写二级域名处填写一个独一无二的域名，接着点击“安装到以上位置”

## 设置站点信息
稍等一会，程序会自动安装的，安装完毕后点击提示中的"点击此处进入初始化页面"进行个人博客的初始化，站点名字，管理员及密码等，设置好了以后，点击下面的“安装wordpress”，一切OK了，个人博客搭建完毕。简单吧。


## 绑定自己的域名
绑定自己的域名，在自己的应用列表下选择刚才你创建的应用名字，不是后面的二级域名啊，切换到“应用设置”里面，在独立域名设置那添加一个你自己的域名，这里添加的时候是把自己的域名添加一个CNAME解析，再添加一个A解析，具体是什么，会有提示告诉你的~

## wordpress 的模板添加与修改
wordpress 的模板添加与修改，这里比用自己的虚拟主机有点麻烦，因为新浪SAE不允许wordpress后台更新模板，所以与下载插件，所以更改模板的时候有两种方 法，SVN或者用SAE的代码编辑器来编辑，推荐使用SVN，比SAE更改要简单方便 SVN的使用方法参见http://sae.sina.com.cn/?m=devcenter&catId=212 这里就不重复了。添加插件也用同样的方法。

# 简单的调用新浪微博API来更新自己的微博

准备工作，新浪微博账号并且开通新浪微博开发者平台，注册地址 http://open.weibo.com/

具体步骤

* 打开新浪微博开放平台 http://open.weibo.com/ 点击下面的“我是开发者” http://open.weibo.com/development 进入开发者页面，点击创建应用--->选择第一个“站内应用”

* 填写应用名字，应用介绍等信息（随便写的），域名绑定选择默认的“否”，点击“创建” OK

* 转到刚才你创建的应用的详细页面，其实这里的信息只有两个有用，一个是APP Keys，一个是App Secret,记录好这两个值，以后的应用都要用这两个值

* 由于新浪SAE已经集合了最新的PHP版的SDK，可以不用下载，直接用，但有兴趣的同学可以下载来看看人家的方法是怎么实现的，对于学习者来说还是挺有用的~

* 在本地新建三个php文件，名字随便起，我这里起的是getRequestToken.php（用于请求request token），请求request token（用于获得的request token和token secret来初始化SaeTOAuth对象），config.php（可选，主要用于初始化常量，推荐使用）

* 在config.php中写入以下代码
``` php
header('Content-Type: text/html; charset=UTF-8');

define( "WB_AKEY" , '你自己的APP Keys' );
define( "WB_SKEY" , '你自己的App Secret' );
define( "WB_CALLBACK_URL" , 'http://kevinweibo.sinaapp.com/getAccess.php' );//稍后介绍这里怎么填写
```

* 在getRequestToken.php中写入以下代码
``` php
<?php
session_start();
include ('config.php');//我这里是将WB_AKEY和WB_SKEY写入到了一个config.php文件中
include ('saet.ex.class.php');
$auth=new SaeTOAuth(WB_AKEY,WB_SKEY);//your app key and app secret
//这一步是去sina服务器请求request token
$token=$auth->getRequestToken();
//这一步是用request token拼装认证的url,第三个参数是获得用户认证许可后跳转到的url。就是我们在SAE注册应用的二级域名
$url=$auth->getAuthorizeURL($token,true,"http://kevinweibo.sinaapp.com/getAccessToken.php");
$_SESSION['token']=$token['oauth_token'];//记录下来备用
$_SESSION['token_secret']=$token['oauth_token_secret'];

//最后将$url当成一个链接输出到页面中,用户点击了这个链接就会跳转到认证页面
echo "<a href=$url>欢迎光临，请使用微博账号受权登录!</a>";
?>
```
 

5.3 在getAccessToken.php中写入以下代码
``` php
<?php
session_start();
include ('config.php');
include ('saet.ex.class.php');
//利用第一步获得的request token和token secret来初始化SaeTOAuth对象
$auth=new SaeTOAuth(WB_AKEY,WB_SKEY,$_SESSION['token'],$_SESSION['token_secret']);
//去换取真正有用的access token
$accessToken=$auth->getAccessToken($_REQUEST['oauth_verifier'],$_REQUEST['oauth_token']);
//print_r($accessToken);
//存储起来,后面每次请求都要带上这两个值
$_SESSION['token']=$accessToken['oauth_token'];
$_SESSION['token_secret']=$accessToken['oauth_token_secret'];
//接下来你就可以跳转到你的应用页面开始微博之旅了,例如输出一句js,window.location.href="send.php"
//echo "token:".$_SESSION['token'];
//echo "<br/>token_secret:".$_SESSION['token_secret']."<br/>";
echo "<script>window.location.href='send.php'</script>"
?>
```

5.4 注意最后一句说的是打开一个send.php页面，所以这里还要新建一个send.php文件，在里面打入以下代码
``` php
<?php
session_start();
include ('config.php');
//include ('saetv2.ex.class.php');
include ('saet.ex.class.php');
$auth = new SaeTClient(WB_AKEY,WB_SKEY, $_SESSION['token'], $_SESSION['token_secret']);
$mes = $_POST[msg];
$auth->update($mes);// 这句话是关键，其实发新浪微博的方法就这个，非常简单~
?>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
</head>
<body>
<form method="POST">
要发送的微博：<input type="text" name="msg">
<input type="submit" value="提交">
</form>
<a href="weibolist.php">进入你的微博列表页面</a><br /> //这个不用写，写了后有更好玩的~
</body>
</html>
```
 

* 把这四个文件打包成一个zip文件，到新浪SAE里面创建一个应用，这时在创建应用的时候就不要选择基于URL安装了，选择下面的“选择文件“，把刚才那压缩的zip文件，点击”安装到以上位置“


* 刚才config.php代码里面的define( "WB_CALLBACK_URL" , 'http://kevinweibo.sinaapp.com/getAccess.php' ); 这里面红字应该知道填写什么了吧？你应用的地址加上getaccess.php

* 好了，大功告成，可以爽一爽了，进入你刚才创建的应用

http://kevinweibo.sinaapp.com/getRequestToken.php

点击上面的受权链接，并将自己的微博账号受权，接着会进入到send.php页面，随便写句话，点击”提交“，看看微博发出去没，哈哈~

# 更多
要玩点更好玩的，可以下载微博SDK，查看里面的API方法，或者在线的查看 http://open.weibo.com/wiki/API%E6%96%87%E6%A1%A3_V2

我这里创建了个weibolist.php文件，来简单的调用了几个
``` php
<?php
session_start();
include ('config.php');
//include ('saetv2.ex.class.php');
include ('saet.ex.class.php');

$auth = new SaeTClient(WB_AKEY,WB_SKEY, $_SESSION['token'], $_SESSION['token_secret']);
$publicmsg=$auth->followers();//调用followers方法来显示当前用户的粉丝列表

 //用foreach方法来遍历数组
foreach ($publicmsg as $key=>$value){
echo "$key => $value"."<br>";
foreach($value as $key1=>$value1){
echo "$key1 => $value1"."<br>";
}
echo "<br>";
}

//这里是旧版的SDK的例子
$mes = $_POST[msg];
$auth->update($mes);
?>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
</head>
<body>
<form method="POST">
要发送的微博：<input type="text" name="msg">
<input type="submit" value="提交">
</form>
<br>
<h2>第二种种发送新微博</h2>
<form action="weibolist.php" method="POST">
<input type="text" name="msg" style="width:300px" />
 <input type="submit" />
</form>

<h2>发送图片微博</h2>
<form action="weibolist.php" >
<input type="text" name="msg" style="width:300px" value="文字内容" />
<input type="text" name="pic" style="width:300px" value="图片url" />
 <input type="submit" />

</form>
<?php
if( isset($_REQUEST['pic']) ){
$rr = $auth ->upload( $_REQUEST['msg'] , $_REQUEST['pic'] );
echo "<p>发送完成</p>" ;
}

?>
```

先到这吧，更有意思的东西等待更新的发现~