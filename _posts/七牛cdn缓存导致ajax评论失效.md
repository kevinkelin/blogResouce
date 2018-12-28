title: 七牛cdn缓存导致ajax评论失效
tags:
  - wordpress
id: 1151
permalink: 1151
categories:
  - 计算机相关
date: 2014-10-07 00:41:23
---

在使用了七牛的CDN后，发现在评论的时候会出现405错误

> {"error":"get from image source failed: E405"} 
 

在网上查了查，这个问题出现的还挺多的，解决办法是改<span style="color: #000000;">comments-ajax.js这个文件</span>

1.在comments-ajax.js文件中下面这段程序的后面（参考[cdn缓存导致ajax评论失效](http://www.tennfy.com/503.html)）：

``` javascript
var i = 0, got = -1, len = document.getElementsByTagName('script').length;
while ( i <= len && got == -1){
    var js_url = document.getElementsByTagName('script')[i].src,
            got = js_url.indexOf('comments-ajax.js'); i++ ;
}
```

<!-- more -->

添加一句话：
> js_url = js_url.replace('static.yangyanxing.com',www.yangyanxing.com');

其中，chengchunjie.qiniu.com换成自己的七牛云存储提供的二级域名，chengchunjie.com替换成自己的域名。

2.登陆七牛云存储，进入自己的网站静态存储的空间，打开内容，找到自己已经缓存的原来的comments-ajax.js文件，将修改过的comments-ajax.js上传上去。（这个最主要是解决，自己网站上comments-ajax.js更改，但七牛的CDN上并没有更改导致的一段时间评论还是无法提交成功的问题）

注意：上传的时候，在设置路径前缀处，一定要自定义前缀，wp-content/themes/你的主题名称/，否则上传无效。