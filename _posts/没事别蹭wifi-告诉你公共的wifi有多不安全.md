title: 没事别蹭wifi-告诉你公共的wifi有多不安全
id: 1298
permalink: 1298
categories:
  - web
date: 2015-02-12 00:13:33
tags:
---

经常看到一些朋友每到一个地方就先找wifi，没有密码就直接上了！汗……

今天通过两个小工具来显示一下公共wifi的安全隐患（爱蹭网的代价）

# 工具：wireshark与cain

# 下载：wireshark
  >[https://www.wireshark.org/download.html](https://www.wireshark.org/download.html "https://www.wireshark.org/download.html")
  > Cain & Abel [http://www.oxid.it/cain.html](http://www.oxid.it/cain.html "http://www.oxid.it/cain.html") (被墙) 百度网盘链接：[http://pan.baidu.com/s/18kshg](http://pan.baidu.com/s/18kshg) 密码：9owu
  > 
  > md5:EA2EF30C99ECECB1EDA9AA128631FF31 sha1:82407EAF6437D6956F63E85B28C0EC6CA58D298A  

如果没有校验工具，我写了一个python脚本来校验 链接：[http://pan.baidu.com/s/1i3j93sp](http://pan.baidu.com/s/1i3j93sp) 密码：h6gz
<!-- more -->
# 使用Cain进行arp欺骗

&#160;&#160;&#160; 打开Cain主界面，先点击上面的小网卡标识，然后再点击那个’+’ 号

[![image](/image/2015/02/image.png "image")](/image/2015/02/image.png) 

默认就行了，它会根据网关进行扫描活动的主机

[![image](/image/2015/02/image1.png "image")](/image/2015/02/image1.png) 

&#160;

可以看到，在和我电脑同网段的有两个三星手机（其中一个是我的）和一台电脑

[![image](/image/2015/02/image2.png "image")](/image/2015/02/image2.png)

&#160;

点击下面的arp图标

[![image](/image/2015/02/image3.png "image")](/image/2015/02/image3.png)&#160; 

接着先点击上面的列表的空白地方，再点击’+ ’号，左边选择要监听的IP，右边选择网关的IP，这里是192.168.1.1（也就是路由器的地址）

[![image](/image/2015/02/image4.png "image")](/image/2015/02/image4.png) 

点击OK，回到主界面，这时点击那个小雷达标识开始进行arp欺骗

[![image](/image/2015/02/image5.png "image")](/image/2015/02/image5.png) 

# 打开wireshark进行抓包

选择capture->interfaces…

[![image](/image/2015/02/image6.png "image")](/image/2015/02/image6.png) 

选择进行监听的网卡（也就是你进行上网的网卡），点击strat

# 抓包分析

我这里抓了我手机上网的包，刷了下微博，看了眼朋友圈，并且用手机里的浏览器访问了我的个人博客[http://static.yangyanxing.com](http://static.yangyanxing.com) 并且登录了下后台

现在来看看这些信息

在wireshark的显示过滤里写下过滤ip.addr eq 192.168.1.102 && !dns && http

找啊找，找到一条

125&#160;&#160;&#160; 2015-02-11 21:25:25.275021000&#160;&#160;&#160; 192.168.1.102&#160;&#160;&#160; 180.149.139.248&#160;&#160;&#160; HTTP&#160;&#160;&#160; 916&#160;&#160;&#160; POST /2/push/active?c=android&i=5f61746&s=2602e25d&ua=samsung-GT-I9502__weibo__4.6.2__android__android4.4.2&wm=4209_8001&v_f=2&from=1046295010&gsid=4up8f9e93Tw5PZz2EqMCx8xEv9p&lang=zh_CN&skin=default&oldwm=2468_1001 HTTP/1.1&#160; (application/x-www-form-urlencoded)

看下它的packet details

打开HTML Form URL Encoded: application/x-www-form-urlencoded，很多信息，但其实没太多的用，顶多那个uid:2035987583 这个是我的微博ID，

打开[http://weibo.com/2035987583](http://weibo.com/2035987583 "http://weibo.com/2035987583") 就到达了我的微博主页了（如果你是个PLMM的话，闷骚的屌丝会通过这样的方式搞到你的微博账号）

[![image](/image/2015/02/image7.png "image")](/image/2015/02/image7.png) 

如果得到微博账号不算什么的话（毕竟微博还算是个开放的平台，加个粉对于博主来说也不算什么坏事），那么如果能够看到你朋友圈的图片呢？

继续往下找抓包信息

会看到这样的包

[![image](/image/2015/02/image8.png "image")](/image/2015/02/image8.png) 

打开这个包的Hypertext Transfer Protocol

拖到下面

[![image](/image/2015/02/image9.png "image")](/image/2015/02/image9.png) 

试着在浏览器里打开这个网址，这个地址就是你朋友圈里朋友发的图片……（汗！）

如果可以看到朋友圈里的图片也无所谓的话，那么如果能看到你登录的用户与密码呢？

继续往下看抓到的包，看到我登录我个人博客后台的包

11695&#160;&#160;&#160; 2015-02-11 21:27:19.177156000&#160;&#160;&#160; 192.168.1.102&#160;&#160;&#160; 118.123.116.226&#160;&#160;&#160; HTTP&#160;&#160;&#160; 121&#160;&#160;&#160; POST /wp-login.php?redirect_to=http%3A%2F%2Fstatic.yangyanxing.com%2F%3Fp%3D1266 HTTP/1.1&#160; (application/x-www-form-urlencoded)

[![image](/image/2015/02/image10.png "image")](/image/2015/02/image10.png) 

打开packet detail 里面的HTML Form URL Encoded: application/x-www-form-urlencoded

[![image](/image/2015/02/image11.png "image")](/image/2015/02/image11.png) 

username 与password已经明文获得到了。。。

# 更多
如果仔细分析包，还能分析出更多有用的信息，在进行arp过程中，手机上没有任何异常，这下知道免费蹭网有多大的隐患了吧？所以没事别想着瞎蹭网，除非你对你要连接的wifi充分的了解与信任。