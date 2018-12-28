title: 与wordpress垃圾评论斗争到底
id: 1069
permalink: 1069
categories:
  - PHP
date: 2014-06-09 23:05:11
tags:
---

由于本人个人博客小站的空间与mysql空间有限，共享服务器资源也有限，所以这些垃圾评论一定要拦截在写入数据库之外。

更可气的是看到空间的统计，很几千的IP访问，但基本上都是这个垃圾评论的IP，所以实在不能忍

网上找了一些方法，主要有三个，如果将这三个一块使用基本上就可以拦截绝大多数垃圾评论
 <!-- more -->
# 使用Akismet插件
后台申请一个免费的key，可以拦截99%的垃圾评论，但是这个插件还是会将垃圾评论写入数据库的，一会几千条甚至上万条垃圾评论，虽说可以设置15天后自动删除，但是看着也别扭，于是加上第二种方法。

# 更改wp-comments-post.php名称
这种方法是先将wp-comments-post.php改为随便的名字，然后在你主题文件的comments.php中将wp-comments-post.php改为你刚才改过的名字
但是现在的spam机器人太强了，这种方法基本上没有任何作用，垃圾评论还是会进入到后台的垃圾评论里，于是采取第三种方法，过滤掉纯英语的垃圾评论，这种评论占了垃圾评论的很大一部分  

# 过滤纯英文垃圾评论
在你主题的functions.php中添加以下代码
``` php
     function refused_spam_comments( $comment_data ) {
     $pattern = '/[一-龥]/u';
     if(!preg_match($pattern,$comment_data['comment_content'])) {
     wp_die('评论必须含中文！');
 }
     return( $comment_data );
 }
```

其中[一-龥]这个正则表达式代表所有中文，这时纯英文的垃圾评论算是过滤掉了，但是还有另外一种垃圾评论，日文的……
 
日文的采用同样的方法，请几个常出现的日文假名写入到正则表达式中ッ、の、ン、優、業、グ、貿 
``` php
   function fuckjp_comment_post( $incoming_comment ) {
     $http = '/[<|=|.|友|夜|KTV|ッ|の|ン|優|業|グ|貿|]/u';
     if(preg_match($http, $incoming_comment['comment_content'])) {
     wp_die( "小日本滚蛋!" );
   }
    return( $incoming_comment );
 }
```

将这三点综合起来使用，终于清净多了……

# 参考文章

[http://www.v7v3.com/wpjiaocheng/201308215.html](http://www.v7v3.com/wpjiaocheng/201308215.html)

[http://www.pzboy.com/soft/php/english-comments/](http://www.pzboy.com/soft/php/english-comments/)