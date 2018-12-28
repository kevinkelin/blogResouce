title: 解决新浪开放平台应用（oauth2）网站无法认证的问题
id: 407
permalink: 407
categories:
  - PHP
date: 2012-11-29 23:10:50
tags:
---

前些天一直在倒腾新浪微博的开放平台，一开始用的oauth1，利用官方的PHP SDK写了一个简单发微博的应用，但是很遗憾，没有成功，这里简单的写一下
``` php
<?php

session_start();
include_once( 'config.php' );
include_once( 'weibooauth.php' );
$c = new WeiboClient( WB_AKEY , WB_SKEY , $_SESSION['last_key']['oauth_token'] , $_SESSION['last_key']['oauth_token_secret'] );

?>

<form action="" method="post">
<input type="text" name="text" id="1">
<input type="submit" name="submit" value="发表微博">
</form>
<?php
if(isset($_POST['text'])){
$rr=$c->update($_POST['text']);

echo "发表成功";

}

?>
```
<!-- more -->
认证页面我就不写了，就是用的官方的SDK，输入几个字以后，点击发表微，虽然显示 “发表成功”，但我的微博根本没有发表，于是我将$rr显示出来
``` php
if(isset($_POST['text'])){
$rr=$c->update($_POST['text']);
 echo "<pre>";
print_r($rr);
 echo "</pre>";

}
```
结果发现了问题
<pre>Array
(
    [request] => /statuses/update.json
    [error_code] => 401
    [error] => 40109:consumer_key_refused!
)</pre>
查看官方文档

"24、OAuth1.0授权方式能否继续使用？

*   目前未授权的应用已经禁止使用OAuth1.0授权及V1接口，2012年9月左右将禁止所有应用使用OAuth1.0授权及V1接口。我们推荐更安全、稳定的OAuth2.0授权方式， OAuth1.0授权方式不久后将全面禁止使用。
"

也就是说oauth1的接口新的应用是不能用的。。。汗，使用oauth2的接口吧

结果却有新的错误

![fq ](http://ww1.sinaimg.cn/bmiddle/795ab47fjw1dz36h36b9mj.jpg)

再继续查看官方文档关于redirect_url_mismatch的解释

“获取用户授权是出现 error ：redirect_uri_mismatch 怎么解决？

*   这是由于回调地址填写错误造成的，解决办法：
*   A、站内应用：redirect_uri等于应用信息中的“站内应用地址”而非“应用实际地址”；
*   B、其他应用：redirect_uri需与[http://open.weibo.com/apps/30871*****/info/advanced](http://open.weibo.com/apps/30871*****/info/advanced) （30871*****替换成你应用的AppKey）应用高级信息中的“应用回调页”完全匹配或在绑定的域名下。
*   注意：修改应用回调页或绑定域名后需要约半小时左右时间生效。
”

由于我新建的是其他应用，于是去修改应用高级设置里的回调页

![](http://ww4.sinaimg.cn/large/795ab47fjw1dzcb580frbj.jpg)

一开始这个回调信息应该是没有的，需要手工输入才可以，于是输入完这个地址后，再来调用此应用，经过受权页面后，一切就OK了~