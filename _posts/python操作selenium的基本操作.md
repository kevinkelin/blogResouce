title: python操作selenium的基本操作
id: 1085
permalink: 1085
categories:
  - Python
date: 2014-06-17 22:59:00
tags:
  - python
  - 自动化测试
---
``` python
#coding:utf-8
from selenium import webdriver
from selenium.webdriver.common.action_chains import ActionChains #引入ActionChains鼠标操作类
from selenium.webdriver.common.keys import Keys #引入keys类操作
import time

def s(int):
    time.sleep(int)
browser = webdriver.Chrome()
browser.get('http://www.baidu.com')
print '现在将浏览器最大化'
browser.maximize_window()
text = browser.find_element_by_name('tj_duty').text
print text #打印备案信息

browser.find_element_by_id('kw1').send_keys(u'杨彦星')
print browser.find_element_by_id('kw1').get_attribute('type')
print browser.find_element_by_id('kw1').size #打印输入框的大小
browser.find_element_by_id('su1').click()
time.sleep(3)

print '现在我将设置浏览器为宽480，高800显示'
browser.set_window_size(480,800)
browser.get('http://m.mail.10086.cn')
time.sleep(3)

print '现在我将回到刚才的页面'
browser.maximize_window()
browser.back()
time.sleep(3)

print '现在我将回到之前的页面'
browser.forward()
time.sleep(5)
print '现在我将打开杨彦星的网站进行json搜索'
browser.get('http://static.yangyanxing.com')
browser.find_element_by_xpath(".//*[@id='ls']").send_keys(u'json')
browser.find_element_by_xpath(".//*[@id='header']/div[1]/div/form/input[2]").click()
time.sleep(5)
browser.quit()

browser = webdriver.Chrome()

print '以下将以登录人人网来进行上面的综合应用'
browser.get('http://www.renren.com/SysHome.do')
browser.find_element_by_id('email').clear()#这个是以id选择元素
browser.find_element_by_id('email').send_keys('email')
browser.find_element_by_id('email').send_keys(Keys.BACK_SPACE)
time.sleep(2)
browser.find_element_by_id('email').send_keys('m')
s(2)
browser.find_element_by_id('email').send_keys(Keys.CONTROL,'a')
s(2)
browser.find_element_by_id('email').send_keys(Keys.CONTROL,'x')#剪切掉里面的内容
s(2)
browser.find_element_by_id('email').send_keys(Keys.CONTROL,'v') #重新输入进去
s(2)
browser.find_element_by_name('password').clear()#这个是以name选择元素
browser.find_element_by_name('password').send_keys('password')
#browser.find_element_by_xpath(".//*[@id='login']").click()#这个是以xpath选择元素
browser.find_element_by_xpath(".//*[@id='login']").send_keys(Keys.ENTER) #这里通过点击Enter键来登录
browser.maximize_window()
article = browser.find_element_by_link_text(u'周碧华：社科院出现内鬼意味着什么？')
ActionChains(browser).move_to_element(article).perform()#将鼠标移动到这里，但是这里不好用
ActionChains(browser).context_click(article).perform()
time.sleep(5)

browser.quit()
```

开始接触selenium，其在web自动化上应用非常广，上面是一些最基本的操作，启动浏览器，打开网页，前进与后退，定位元素，键盘输入与鼠标点击操作，其中xpath可以在firefox下应用firepath插件来获取，但是局限性比较大，也会有一些兼容性的问题，像上面的代码在chrome下可以运行，但是在firefox下一些元素就找不到，很郁闷。。。继续再探索吧