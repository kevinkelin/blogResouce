title: 解决wordpress后台写日志时编辑器加载慢
id: 444
permalink: 444
categories:
  - 计算机相关
date: 2012-12-08 00:05:04
tags:
---

空间放在国外的人基本都有一个感受，有时加载很慢，我最近用wordpress写文章的时候，默认的编辑器加载很慢
有时都无法写博客下图中红框圈出来的位置加载太慢，后来网上查了查相应的资料，原来是有一个.js文件导致的，那个文件有200多K
所以有时对于主机放在国外的朋友很慢，其实这里只要把这个js文件放到国内服务器，加载速度就能提高很快

![](http://ww1.sinaimg.cn/large/795ab47fjw1dzlljowwkkj.jpg "reload")

 

到你的wordpress目录 wp-includes/js/tinymce取出tiny_mce.js文件，然后将其放到国内一个访问快的服务器，别说你没有啊，实在没有的话用新浪的SAE。然后找到打开wp-includes/class-wp-editor.php 这个文件，找到大概540行左右
<!-- more -->
改为
``` php
if ( $tmce_on ) {
if ( $compressed )
echo "<script type='text/javascript' src='{$baseurl}/wp-tinymce.php?c=1&$version'></script>n";
else
echo "<script type='text/javascript' src='http://kevinkelin.sinaapp.com/tiny_mce.js'></script>n";

if ( 'en' != self::$mce_locale && isset($lang) )
echo "<script type='text/javascript'>n$langn</script>n";
else
echo "<script type='text/javascript' src='{$baseurl}/langs/wp-langs-en.js?$version'></script>n";
}
```

其中http://kevinkelin.sinaapp.com/tiny_mce.js 是我将这个js文件放到新浪SAE的地址，如果你的版本也是3.4.2的话，你想用也可以引用，前提是你认为我不会对这个js文件做手脚。。。。

 

好了，改好后再在后台写篇文章看一下吧，速度是不是有很大的提高呢？