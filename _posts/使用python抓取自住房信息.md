title: 使用python抓取自住房信息
comment: true
categories:
  - Python
date: 2017-07-11 22:41:32
tags: 
  - python
permalink: use-python-get-house-info

---
使用python也有一段时间了，最近比较关注自住房信息，虽说它更新的比较缓慢，但是平时也不怎么会特意的去它的网站上去看，
于是就想用python抓它的信息，如果有新的信息就给自己发个邮件，这样手机上得到通知以后就可以再去它的网站上看看。
功能比较简单，但是用到的点还是挺多的，这里记录一下。
主要有以下几个步骤
1. python beautifulsoup 与requests的使用
2. ubuntu 中安装 mysql 与mysql-python 
3. beautifulsoup与requests编码的问题
4. 使用gmail发送邮件，其中gmail采用两步认证要单独申请一个密码
5. 在ubuntu中使用crontab定时来触发脚本

<!-- more -->

# 网站分析
自住房信息的网址为 http://www.bjjs.gov.cn/bjjs/fwgl/zzxspzf/tzgg/index.shtml 
主要就是抓取上面的通知，使用数据库或者本地文件记录一下url和标题
主要就是以下这块html
``` html
<ul opentype="page"> 
 <li><a href="/bjjs/fwgl/zzxspzf/tzgg/427793/index.shtml" onclick="void(0)" target="_blank" title="富兴鹏城自住型商品住房递补选房公告">富兴鹏城自住型商品住房递补选房公告</a><span>2017-07-06</span></li>
 <li><a href="/bjjs/fwgl/zzxspzf/tzgg/425403/index.shtml" onclick="void(0)" target="_blank" title="朝阳区锦都家园自住型商品住房项目申购登记公告">朝阳区锦都家园自住型商品住房项目申购登记公告</a><span>2017-06-12</span></li>
 <li><a href="/bjjs/fwgl/zzxspzf/tzgg/425259/index.shtml" onclick="void(0)" target="_blank" title="住总万科&middot;TBD万科天地自住型商品住房递补选房公告">住总万科&middot;TBD万科天地自住型商品住房递补选房公告</a><span>2017-06-09</span></li>
 ....
 ...
 ..
 .
</ul>
```
我这里使用的是beautifulsoup来进行解析。
beautifulsoup的使用参考 [beautifulsoup使用](http://cuiqingcai.com/1319.html)
我一开始使用requests库的get方法来抓取网页，后来我被它的编码逼疯了，在网上查了下，requests与beautifulsoup都会对网页的编码进行优化，

![乱码](/image/houseluanma.png)
但是它们同时优化就会出现很多头疼的问题，所以最后我使用python的urllib来抓取网页

``` python 
url = r'http://www.bjjs.gov.cn/bjjs/fwgl/zzxspzf/tzgg/index.shtml'
s = urllib.urlopen(url)
if s.code == 200:
	soup = BeautifulSoup(s.read(),"lxml")
```

## 解析出url 与title信息
``` python
ulcontent = soup.find('ul',opentype="page")	
for i in ulcontent.children:
	acontent = i.find('a')
	if type(acontent)!=int:
		title = acontent.get('title').encode('utf-8')
		if acontent.get('href').startswith(r'/'):
			url = "%s%s"%(r'http://www.bjjs.gov.cn',acontent.get('href'))						
		else:
			url = acontent.get('href')

```

使用 `soup.find('ul',opentype="page")` 来定位到通知的ul dom结构，然后再寻找下面的a结构，美汤的用法还是很牛逼的。

# 使用mysql来记录

得到url与title信息后，就去数据库中查寻一下，我一开始想用mongo，但是由于我的VPS是OpenVZ的，所以安装mongodb以后发现有很多问题，最后索性放弃，改用mysql。
ubuntu 上安装mysql还是挺简单的
> sudo apt-get install mysql-server mysql-client

安装过程中会设置root密码，安装结束后，我创建了一个库和表
``` html
mysql> CREATE DATABASE houseinfo；
mysql> use houseinfo;
mysql> create table house ( id int NOT NULL AUTO_INCREMENT PRIMARY KEY, url text NOT NULL, title text NOT NULL) DEFAULT CHARSET=utf8;

```

在表中查询url字段是否存在,不存在的话就插入
``` python 
def executesql(sql):
	dbo  = MySQLdb.connect("localhost","root","password","houseinfo")
	cursor = dbo.cursor()
	data = ''
	try:
		cursor.execute(sql)
		if sql.startswith('select'):
			data = cursor.fetchall()
		elif sql.startswith('insert'):
			data = dbo.commit()
	except:
		print "Error: unable to fecth data"
		print traceback.format_exc()
		dbo.rollback()
	finally:
		dbo.close()		
		return data

s = urllib.urlopen(url)
if s.code == 200:
	soup = BeautifulSoup(s.read(),"lxml")
	ulcontent = soup.find('ul',opentype="page")	
	for i in ulcontent.children:
		acontent = i.find('a')
		if type(acontent)!=int:
			title = acontent.get('title').encode('utf-8')
			if acontent.get('href').startswith(r'/'):
				url = "%s%s"%(r'http://www.bjjs.gov.cn',acontent.get('href'))						
			else:
				url = acontent.get('href')

			sql = 'select * from house where url = "%s"'% url
			rst = executesql(sql)
			if not rst:
				insertsql = 'insert into house (url,title) values("%s","%s")'%(url,title)
				executesql(insertsql)

```

# 发送新通知邮件

当有一个新的通知发出来以后需要发送一封邮件

因为VPS是在国外，所在我这里使用的是gmail的smtp服务，因为我的google账户采用了两步认证，所以这里还得重新申请一个临时密码，这里不得不提一下google的服务，太人性化了！具体使用请参考 https://support.google.com/accounts/answer/185833

``` python 
HOST = 'smtp.gmail.com'
PORT = 587
mail_username='yanxingyang@gmail.com'  
mail_password='linshimima'

to_addrs=['yanxingyang@gmail.com'] 

def sendmail(contents,title):
	try:
		smtp = smtplib.SMTP() 
		smtp.set_debuglevel(1)
		smtp.connect(HOST,PORT)
		smtp.starttls()
		smtp.login(mail_username,mail_password)
		msg = email.mime.text.MIMEText(contents,'html')
		msg['From'] = 'yanxingyang@gmail.com'
		msg['To'] = ';'.join(to_addrs)  
		msg['Subject']=  '有新的房产信息,请注意查看《%s》'% title   
		smtp.sendmail('yanxingyang@gmail.com',to_addrs,msg.as_string())  
	except:
		print traceback.format_exc()
	finally:
		smtp.quit()

```

# 设置crontab任务

在linux的世界里crontab绝对是神一样的存在
使用参考 [crontab使用](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)

我这里设置的是每隔一个小时执行一次
```
* */1 * * * python /root/gethouseinfo.py
```

最终的效果是我收到了好多邮件。。。 
![手机中收到的邮件](/image/houseinfo.png)

mac上也可以实时的收到
![mac上的邮件](/image/housemac.png)

最张的全代码
``` python 
#coding:utf-8

import time,os,sys,chardet
from datetime import datetime
import requests
from bs4 import BeautifulSoup
import json
import MySQLdb
import smtplib
import email.mime.text
import traceback
from email.utils import parseaddr, formataddr
from email import encoders
from email.header import Header
import urllib

HOST = 'smtp.gmail.com'
PORT = 587
mail_username='yanxingyang@gmail.com'  
mail_password='linshimima' 

to_addrs=['yanxingyang@gmail.com'] 

urls = [r'http://www.bjjs.gov.cn/bjjs/fwgl/zzxspzf/tzgg/index.shtml']

def sendmail(contents,title):
	try:
		smtp = smtplib.SMTP() 
		smtp.set_debuglevel(1)
		smtp.connect(HOST,PORT)
		smtp.starttls()
		smtp.login(mail_username,mail_password)
		msg = email.mime.text.MIMEText(contents,'html')
		msg['From'] = 'yanxingyang@gmail.com'
		msg['To'] = ';'.join(to_addrs)  
		msg['Subject']=  '有新的房产信息,请注意查看《%s》'% title   
		smtp.sendmail('yanxingyang@gmail.com',to_addrs,msg.as_string())  
	except:
		print traceback.format_exc()
	finally:
		smtp.quit()

def executesql(sql):
	dbo  = MySQLdb.connect("localhost","root","password","houseinfo")
	cursor = dbo.cursor()
	data = ''
	try:
		cursor.execute(sql)
		if sql.startswith('select'):
			data = cursor.fetchall()
		elif sql.startswith('insert'):
			data = dbo.commit()
	except:
		print "Error: unable to fecth data"
		print traceback.format_exc()
		dbo.rollback()
	finally:
		dbo.close()		
		return data

def main():
	for url in urls:
		# r = requests.get(url)
		s = urllib.urlopen(url)
		if s.code == 200:
			soup = BeautifulSoup(s.read(),"lxml")
			ulcontent = soup.find('ul',opentype="page")	
			for i in ulcontent.children:
				acontent = i.find('a')
				if type(acontent)!=int:
					title = acontent.get('title').encode('utf-8')
					if acontent.get('href').startswith(r'/'):
						url = "%s%s"%(r'http://www.bjjs.gov.cn',acontent.get('href'))						
					else:
						url = acontent.get('href')

					sql = 'select * from house where url = "%s"'% url
					rst = executesql(sql)
					if not rst:
						insertsql = 'insert into house (url,title) values("%s","%s")'%(url,title)
						executesql(insertsql)
						contents = '<h1>%s</h1></br><p>%s</p>'%(title,url)
						sendmail(contents,title)

if __name__ == '__main__':
	main()

```

# 遗留的问题
如果长时间的使用某个IP去抓该网站的信息它如果给封了IP就不行了，而且我也没有设置访问的header，以后可以设置一下代理或者header信息





