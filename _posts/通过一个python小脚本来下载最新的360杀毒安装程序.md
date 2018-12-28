title: 通过一个python小脚本来下载最新的360杀毒安装程序
id: 817
permalink: 817
categories:
  - Python
date: 2013-06-03 19:21:12
tags:
  - python
---

小脚本很简单，主要是练习一下正则匹配，与简单的urllib库的应用

``` python
#coding: utf8
import urllib,re
import os

def getLastinstall():

    page = urllib.urlopen(r'http://sd.360.cn/download_center.html')
    html = page.read()
    page.close()
    rematch = r'http://down.360.cn/360sd/360sd_se_(.*?)exe'
    replusmatch = r'http://down.360.cn/360sd/360sd_plus_(.*?)exe'
    stdname = '360sd_'+(str(re.compile(rematch).findall(html)[0]))+'exe'
    downurlstd = r'http://down.360.cn/360sd/360sd_'+(str(re.compile(rematch).findall(html)[0]))+'exe'
    powname = '360sd_plus_'+(str(re.compile(replusmatch).findall(html)[0]))+'exe'
    downurlpow = r'http://down.360.cn/360sd/360sd_plus_'+(str(re.compile(replusmatch).findall(html)[0]))+'exe'
    localpath = os.getcwd()+r'/installpack'
    if not (os.path.isfile(stdname) and os.path.isfile(powname)):
        if not os.path.isdir('installpack'):
            os.makedirs('installpack')
        urllib.urlretrieve(downurlstd,localpath+'/'+stdname)
        urllib.urlretrieve(downurlpow,localpath+'/'+powname)
    else:
        print '目录中已经是线上最新版杀毒安装程序'

if __name__ =='__main__':
    getLastinstall()
```