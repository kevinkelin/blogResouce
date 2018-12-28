title: 在wireshark上显示host
comment: true
categories:
  - 计算机相关
date: 2016-06-10 22:19:45
tags: 
  - 安全
permalink: display-host-clumn-in-wireshark

---

wireshark默认不显示目标的host，只显示IP，有时对于查看非常不直观。
可以自定义显示一些列
![wireshark](/image/wireshark.jpg)

点击"编辑"-->"首选项"
![首选项](/image/wiresharkOption.png)

点击下面的"+"，添加一个列，类型选择Custom,字段那填写http.host,字段发生那填写0,点击OK，界面上就会显示host字段了
![显示host](/image/showhost.png)
