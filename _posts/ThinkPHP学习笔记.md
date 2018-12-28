title: ThinkPHP的学习笔记
comment: true
categories:
  - PHP
date: 2016-01-30 20:38:08
tags: 
  - PHP
  - ThinkPHP
permalink: thinkphp-study-note

---

# 初始化一个项目

thinkphp是单入口文件
``` php
<?php
define("APP_NAME", "THINK_Study");
define("APP_PATH", "./study/");
define("APP_DEBUG", true);
require("./ThinkPHP/ThinkPHP.php");
```

Note:
1. 要先定义APP_PATH,后再include thinkphp 主入口文件，要不然会在当前目录上建立目录，很乱
2. 各种版本生成的目录有些不同，我现在用的是3.2.3 就没有lib目录，而是只有一个home目录,下面的Controller目录相当于之前版本的是lib目录，里面记录着控制器，是MVC中的C，同级目录还有Model和View目录，这样更明显
3. 各个项目下的`Conf\config.php` 里的内容只有自已的项目才能读取，其它项目读取不了，比如`admin`读不了`study`配置
<!-- more -->
# 关于多模块的设计

1. 3.2.3以后采用多模块设计，即一个项目里采用多个模块的设计思路，比如可以有一个前台的web模块，再加上一个后台的admin模块，而之前版本的thinkphp则需要建立两个入口文件 index.php=>web,admin.php=>admin
2. 多模块后，则要url上加上模块名，如 http://localhost/index.php/admin/index/index
3. 可以绑定一个默认的模块，在主入口文件中加入
`define('BIND_MODULE','Home');`则将home模块绑定到默认模块中，这样在访问home模块的时候就可以不用加上home了，如 http://localhost/index.php/index/index 则是访问home模块下的index控制器下的index方法
4. 可以在项目的`Config.php`文件中自定义模块的访问，达到一种不写Home模块时使用Home模块，写上其他的模块则使用其它的模块,`不能写在index.php中，没用的`
> 'MODULE_ALLOW_LIST'    =>    array('Home','Admin','User'),
  'DEFAULT_MODULE'       =>    'Home',

5. 我曾尝试一次性生成多个模块目录，但是按照官方的方法不能实现


# 使用U方法生成url
[ThinkPHP函数详解：U方法](http://www.thinkphp.cn/info/132.html)
> $url = U("Index/index@sina.com",array('id' => 1),"html",false);

# 使用公共库函数
在3.2 版本中，不再是common.php，要在项目的Common目录下建立一个function.php 

# 数组在模板中的显示
1. 一维数组
``` php
$jin = array(
            'name' => 'zhangjin', 
            'sex'  => 'female',
            'age'  => '25',
            );
$this->assign("jin",$jin);
$this->display();
```

上面定义了一个一维数组，将其赋值于jin,然后渲染到对应的模板中
我个人比较喜欢用foreach

``` html
<foreach name="jin" item='vo'>
{$key}---{$vo}<br />
</foreach> 
```

`$key` 为一维数组中的key

2. 多维数组在模板中的遍历
其实更多的时候是使用的多维数组，很多时候在数据库中取得数据以后在模板中进行渲染

``` php
$psersons = array(
    1=>array('name'=>'yyx','age'=>28,'sex'=>'male'),
    2=>array('name'=>'jin','age'=>24,'sex'=>'female'),
    3=>array('name'=>'shi','age'=>28,'sex'=>'female'),
    )
$this->assign('persons',$persons);
$this->display();
```

``` html
<foreach name="persons" item='person'>
{$person['name']}---{$person['age']}---{$person['sex']} <br />
</foreach>
```

它是将二维数组里第一组作为一项进行遍历

# CURD

1. M方法，大M方法可以在不实现模型的基础上对数据库进行简单的curd，多以查询为主
> $data = M('user') 

2. 查询方式
2.1 使用字符串作为查询条件 
> $User = M("User"); // 实例化User对象
$User->where('type=1 AND status=1')->select(); 

生成的SQL语句是
> SELECT * FROM think_user WHERE type=1 AND status=1

2.2 使用数组作为查询条件
> $data = M('data');
$condition['id'] = 1;
$condition['data'] = 'framework';
$condition['_logic'] = 'or';
$rst = $data->where($condition)->select();

生成的SQL语句是
> SELECT * FROM `think_data` WHERE `id` = 1 OR `data` = 'framework'

注意
1. 如果数组里的key不是表中的列的话则该条件不生效
2. `_logic` 默认是`and`

2.3 表达式查询
这种查询比较灵活

表达式 含义
EQ  等于（=）
NEQ 不等于（<>）
GT  大于（>）
EGT 大于等于（>=）
LT  小于（<）
ELT 小于等于（<=）
LIKE    模糊查询
[NOT] BETWEEN   （不在）区间查询
[NOT] IN    （不在）IN 查询
EXP 表达式查询，支持SQL语法

`$where[字段名]=array(表达式,查询条件)`

> $condition['id'] = array('lt',3);
$rst = $data->where($condition)->select();

SQL: SELECT * FROM `think_data` WHERE `id` < 3 

> $condition['id'] = array('lt',3);
$condition['data'] = array('like','%php');
$rst = $data->where($condition)->select();

SQL: SELECT * FROM `think_data` WHERE `id` < 3 AND `data` LIKE '%php' 

如果想要查询以php结尾或者是fr开头的
>$condition['id'] = array('lt',3);
$condition['data'] = array('like',array('%php','fr%'));
$rst = $data->where($condition)->select();

SQL: SELECT * FROM `think_data` WHERE `id` < 3 AND (`data` LIKE '%php' OR `data` LIKE 'fr%')

2.4 区间查询
> $condition['id'] = array(array('gt',0),array('lt',3),'or');
$rst = $data->where($condition)->select();  

SQL: SELECT * FROM `think_data` WHERE ( `id` > 0 OR `id` < 3 )

区间查询与上面的表达式查询中的between有一些相似,但是`between`只能是`and`的关系，没有区间查询中的`or`关系

> $condition['id'] = array('between','1,5');
$rst = $data->where($condition)->select();

SQL: SELECT * FROM `think_data` WHERE `id` BETWEEN '1' AND '5' 

