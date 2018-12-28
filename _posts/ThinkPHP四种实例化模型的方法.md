title: ThinkPHP四种实例化模型的方法
id: 372
permalink: 372
categories:
  - PHP
date: 2012-08-07 01:00:18
tags:
  - ThinkPHP
---

假设数据库中有一个think_user的数据表

# 普通方式
``` php
$user=new Model('user');

$user=M('user');

/*

user的U可以大写，也可以小写，当库中还有一个think_user_message等表的时候，可以使用

new Model('userMessage') 或者new Model(user_message)，也就是说大写的首字母相当于带下划线的单词

此方法也可以用快捷方法，M方法

*/
```
<!-- more -->
# 当需要继承多个模型的时候，可以用M(' ',' ')方法
``` php
$user=M('user','Common');
$user=new Common('user');

/*

此方法说明$user中既有user中的方法，也可以从自定义的模型中继承方法，比如在Common中写一个常用的方法（公共方法），

这种方法比较便捷的实现了一个变量实例化多个模型，也大大的简便了代码的书写,此方法还可以写成

$user=new Common('user');

*/
```
 

# D方法
``` php
$user=new UserModel();
$user=D('user');
/*
此种方法可以方便实现用户自定义数据库操作类与方法，用户可以在UserModel.class.php中封装自己定义的方法
*/
```

# 通过空模型来实例化
这种方式是通过自定义数据库查询语句来实现查询功能
``` php
$user=new Model();
$list=$user->query('select * from think_user')
```
总的来说，如果没有继承第三方的类，D与M方法实例出来的是一样的

$user=M('user');

$user=D('user');

而M方法则可以比较方便的继承第三方类（公共类啊等等）