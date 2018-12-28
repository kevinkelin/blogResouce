title: 一个Python的小例子来理解字典与while循环
id: 393
permalink: 393
categories:
  - Python
date: 2012-11-12 01:23:59
tags:
  - python
---

现在要实现一个需求，弹出一个界面，可以让用户选择是新建用户还是登录已有账户，或者直接退出，在登录账户的时候，密码输入错误不能超过三次，超过三次要回到主界面，用户名输入‘q’的时候也可以退出并回到主界面
<!-- more -->
``` python
#-*-coding:utf-8-*-

db = {}  #通过使用字典来建立一个以用户名与密码的映射关系
def newuser():
    prompt = 'login desired:'
    while True:  #开始第一层循环，一直是True，除非有break
        name = raw_input(prompt)
        if db.has_key(name):  #判断所输入的用户名是否在已经存在的字典中，也就是字典中的key，如果存在则弹出一个提示
            prompt = 'name has already please change another'
            continue
        else:
            break  #这里的意思是所输入的用户名不在字典的key里，则循环break掉
    pwd = raw_input('password: ')
    db[name] = pwd    #将所输入的密码赋值给用户名所对应的value

def olduser():    #这里定义一个老用户登录的函数
    us = False    #这里定义一个us=False的目的是为了进行检查用户名是否存在的
    while not us:
        username = raw_input('username(q for quit): ')
        if username in db:    #这处是检查用户名（key）是否是所定义的字典中，如果在的话us = True，则此处循环结束，不再提示输入用户名
            us = True
            pwded = db.get(username)    #取出用户名的密码
            Ntime = 3    #定义最多只能错误输入三次密码
            while Ntime != 0:
                password = raw_input('password: ')
                if password != pwded:
                    print 'You password is not correct you have %d times to try again' % Ntime
                    Ntime-=1    #次数减1
                    if Ntime == 0:
                        print 'You have tried too much time without right password!'
                        break    #循环结束，同时也意味着olduser()函数的结束，程序向下走，程序将调用__main__入口，从而showmenu()函数被调用

                elif password == pwded:    #密码正确
                    print 'welcome back!'
                    break

        elif username == 'q':    #当用户名输入q 的时候手工定义pwded and password，为了后面的判断
            pwded = None
            password = 123
            break
        else:
            print 'No such user!!! please try again'    #当输入一个不存在的key的时候输入此句

    if password == 123:
        print 'You choice is q so it will back to main menu!'    #这里是前面的当输入为q的用户名时的输出

def showmenu():
    prompt = '''
    (N)ew user sign
    (O)ld user login
    (Q)uit
    Enter your choice:'''
    done = False
    while not done:
        chosen = False
        while not chosen:
            try:
                choice = raw_input(prompt).strip()[0].lower()    #取用户所输入字符串第一个字符并将其小写去空格
            except (EOFError,KeyboardInterrupt):
                choice = 'q'
            print 'nYou picked is [%s]'% choice
            if choice not in 'noq':    #判断是否在这三个字母当中
                print 'choice is invaild,try again'
            else:
                chosen = True

        if choice == 'q':done =True    #done = True 则程序主循环不执行，程序结束
        if choice == 'o':olduser()
        if choice == 'n':newuser()

if __name__ == '__main__':
    showmenu()
```