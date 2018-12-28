title: 使用wireshark抓取解密https包
comment: true
categories:
  - 计算机相关
date: 2016-06-11 22:40:07
donate: true
tags: 
  - 安全
permalink: use-wireshark-capture-https

---

今天在看https的相关技术，于是想要使用wireshark抓取并解密https流量
关于https的基础可以看看这篇文章
[HTTPS理论基础](http://www.yangyanxing.com/article/https-basic.html)

本文参考文章
[使用 Wireshark 调试 HTTP/2 流量](https://imququ.com/post/http2-traffic-in-wireshark.html)
<!-- more -->

当系统环境变量中存在`SSLKEYLOGFILE`这个变量的时候，chrome与firefox在访问https网站的时候会将密钥写入这个环境变量中，如果在wireshark中设置一下，这样就可以解析https的流量了。
1. 设置环境变量，添加SSLKEYLOGFILE变量，不同的系统不一样
2. 在wireshark中设置，打开wireshark,点击编辑->首选项，切换到PRotocols，选择SSL，在(Pre)-Master-Secret log filename那选择刚刚SSLKEYLOGFILE的值
![wireshark设置](/image/Setting-wireshark-protols.png)
3. 开始抓包吧
![抓到的访问alipay的流量](/image/alipayhttps.png)

可以看到，wireshark已经将http/2转成了http，并且在下面显示了相应的Decrypted SSL data的tab，我随便输入了用户名与密码，已经可以在wireshark上显示了

