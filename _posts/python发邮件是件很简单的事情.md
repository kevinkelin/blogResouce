title: python发邮件是件很简单的事情
id: 870
permalink: 870
categories:
  - Python
date: 2013-08-08 00:09:33
tags:
  - python
---

利用python的smtplib，发邮件将是一件非常简单的事情，下文以用163邮箱来发邮件为例，说明smtplib的应用
``` python
#coding:utf-8
import smtplib

def sendMail(mail_to):
    mail_server = 'smtp.163.com'
    mail_port = '25'
    username = 'soar_1987@163.com'
    password = 'XXXXXXXX'
    mail_title = 'python Test'
    mail_content = 'This is a test from python for sending email'
    if type(mail_to) == str:#之所以要有这样的判断是为了收件人是多个人或者传入的的收件人列表是以list的方式
        mail_list = mail_to.split(';') #将str类型的数据转换为list
    elif type(mail_to) == list:
        mail_list = mail_to
    else:
        print "你输入的收件人格式有误"

    try:
        handle = smtplib.SMTP(mail_server,mail_port)
        handle.login(username,password)
        msg = "From:%srn To:%srnContent-Type: text/html;charset=gb2312rnSubject:%srnrn %s"%("杨彦星",";".join(mail_list),mail_title,mail_content) #这里的msg其实就是一种固定的格式，From:To:Subject
        handle.sendmail(username,mail_list,msg)
        print "Send email sucess"
    except Exception,e:
        print "Send email failed because %s" % e

if __name__ =="__main__":
    sendMail('yanxingyang@gmail.com')
    
```