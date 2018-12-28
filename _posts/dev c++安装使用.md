title: dev c++安装使用
id: 1208
permalink: 1208
categories:
  - C++
date: 2014-12-06 22:08:03
tags:
---

最近开始接触C++，在练习基本的语法的时候，不想使用臃肿的VS，于是查了下，dev c++为个挺小巧，但麻雀虽小五脏具全，对于基本语法的学习使用已经足够了。

下载安装，下载还是挺麻烦的，我下载好了共享在360云盘上了，[http://yunpan.cn/cAaf826jiSdie](http://yunpan.cn/cAaf826jiSdie) （提取码：7b0a）这个是有minGW的，安装的时候选择full

在翻译的时候输出debug，当然你不这样设置的话在以后要调试的时候也会提示你是否设置的 工具--翻译选项

<!-- more -->

写一个简单的hello world吧
  ```  cpp
#include<iostream>
using namespace std;

int main(){
cout<<"hello world"<<endl;
return 0;
```

[![QQ截图20141206220554](/image/2014/12/QQ20141206220554_thumb.png "QQ截图20141206220554")](/image/2014/12/QQ20141206220554.png)