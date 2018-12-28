title: 使用python操作selenium操作第三方浏览器(360浏览器)
comment: true
categories:
  - Python
date: 2018-04-26 22:41:32
tags: 
  - python
  - 自动化测试
permalink: use-selenium-op-browser

---

最近在测试一个项目，使用360浏览器来做一些操作
之前使用过selenium来操作chrome和FireFox，这里记录一下

<!-- more -->

环境：windows7 python2.6 webdriver

# 使用chrome进行测试
安装selenium 
> pip install selenium 

下载浏览器驱动(chrome)
[浏览器驱动](https://sites.google.com/a/chromium.org/chromedriver/downloads)

一定要根据自已的chrome版本来下载对应的chromedriver
![download](/image/2018/chromedriver.png)

将chromedriver.exe的路径加到系统的环境变量中，个人用户和系统的都行
简单的测试一下selenium是否工作正常
``` python 
from selenium import webdriver
d = webdriver.Chrome()
d.get(r'http://www.yangyanxing.com')
```

当输入* d = webdriver.Chrome() * 时，如果没有异常，那会将会打开Chrome浏览器
![chrome](/image/2018/chrome.png)

接下来就可以用selenium的接口来进行相应的操作了
selenium的接口文档 
[selenium接口文档](http://selenium-python.readthedocs.io/index.html)

> print d.find_element_by_class_name('text-muted').text.encode('gbk') 
> 京ICP备18004468号

# 使用第三方浏览器进行测试，这里以360安全浏览器为例
首先查看它的chrome内核版本
![360安全浏览器内核版本](/image/2018/360se.png)

它的版本是55的，好低啊，对应的chromedriver.exe是2.28，
到https://chromedriver.storage.googleapis.com/index.html?path=2.28/ 处下载

这里要用到的是实例webdriver.Chrome()时要加上一些参数

``` python
chrome_options = webdriver.ChromeOptions()
chrome_options.binary_location = r"C:\Users\kevin\AppData\Roaming\360se6\Application\360se.exe" #这里是360安全浏览器的路径
chrome_options.add_argument(r'--lang=zh-CN') # 这里添加一些启动的参数
d = webdriver.Chrome(chrome_options=chrome_options)
```

不出意外的话将启动360安全浏览器，之后就可以继续使用selenium的api来操作网页了


# 可能出现的问题

浏览器闪退

查看报错信息
> raise exception_class(message, screen, stacktrace)
selenium.common.exceptions.WebDriverException: Message: session not created exception: Chrome version must be >= 65.0.3325.0

这个的意思就是chromedriver.exe版本不对，请下载与chrome内核版本对应的chromedriver.exe

> selenium.common.exceptions.WebDriverException: Message: 'chromedriver' executable needs to be in PATH. Please see https://sites.google.com/a/chromium.org/chromedriver/home

这个的意思是chromedriver.exe 没有在环境变量里，请将chromedrive.exe放到环境变量里即可。

之后将研究一下远程的WebDriver。

参考文章
[Python爬虫利器五之Selenium的用法](https://cuiqingcai.com/2599.html)
[selenium 定制启动 chrome 的选项](https://blog.csdn.net/vinson0526/article/details/51850929)


