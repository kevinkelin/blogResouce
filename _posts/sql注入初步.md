title: sql注入初步
id: 1314
permalink: 1314
categories:
  - web
date: 2015-02-22 18:40:30
tags:
  - 安全
  - sql
---

自已用php+mysql写了一个登录页面。其中包含多个sql注入漏洞，在mysql中创建一个表，其中有username password email一个列，添加三个数据

+-------------+-------------+-----------------+ 
| username    | password    | email           | 
+-------------+-------------+-----------------+ 
| admin       | admin       | admin@admin.com | 
| yangyanxing | yangyanxing | yyx@yyx.com     | 
+-------------+-------------+-----------------+

在phpmyadmin中使用一条sql命令

SELECT * FROM `admin` WHERE 1 

where 1 是一个永真，这样它会把admin表中的所有数据返回

写一个testsql.php文件来尝试使用sql注入的方式登录这个系统<!-- more -->
  ```  php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

<?php
define('DB_HOST', 'localhost');
define('DB_USER', 'root');
define('DB_PWD', '123456');
define('DB_NAME', 'test');

_connect(); //连接数据库
_select_db();//选择数据库
_set_names();//设置字符集

//连接数据库
function _connect(){
//$coon这个变量是全局的，它可以使得其在函数外面也可以访问到。 这里用global修饰
global $conn;
if(!$conn = mysql_connect(DB_HOST,DB_USER,DB_PWD)){
exit('数据库连接失败');
}
}

function _select_db(){
if (!mysql_select_db(DB_NAME)){
exit('找不到指定的数据库');
}
}

function _set_names(){
if (!mysql_query("SET NAMES UTF8")){
exit('设置字符集错误！');
}
}

/**
 * @param $_sql
 */
function _query($_sql){
if(!$checkResult = mysql_query($_sql)){
exit('SQL语句执行失败'.mysql_error());
}
return $checkResult;
}

/**
 *
 * @param unknown $sql
 * @return multitype:
 */
function _fetch_array($sql){
return mysql_fetch_array(_query($sql),MYSQL_ASSOC);
}

/**
 * @return int
 */
function _mysql_affected_rows(){
return mysql_affected_rows();
}


/**
 * @param $username
 * @param $info
 */
function checkUserUni($username,$info){
$sql = "SELECT tg_username FROM tg_user WHERE tg_username='$username' LIMIT 1";
if(_fetch_array($sql)){
go_back($info);
}
}

if(isset($_POST['username']) && isset($_POST['password'])){
    $username = $_POST['username'];
    $password = $_POST['password'];
    $query = "select * from admin where username = '$username' and password='$password'";

    echo $query.'<br>';
    $result = _fetch_array($query);

    if($result){
        echo "登录成功";
    }else{
        echo "登录失败";
    }
    mysql_close();
    print_r($result);

}
?>
<body>
<form action='' name='login' method="post">
    <dd>用 户 名：<input type="text" name="username" class="text" /></dd>
    <dd>密　　码：<input type="password" name="password" class="text" /></dd>
    <dd><input type="submit" value="登录" class="button" /></dd>
</form>

</body>

```

其中

$query = "select * from admin where username = '$username' and password='$password'"; 

这里的$username与$password未经过任何过滤，而是用户输入什么就是什么

首先在用户名里输入一个单引号（'） 来看看返回什么错误

You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '''' and password=''' at line 1

如果有类似于这样的错误则说明程序没有将单引号过滤掉，可能有sql注入漏洞,接着再看sql查询语句

$query = "select * from admin where username = '$username' and password='$password'"; 

只要构造一个where后面是一个永真的条件就可以了，由于后面用了一个and逻辑，要构造两边都为true才行

username=’’ 可以构造一个username=’’ or ‘1’ = '1’ 

password=’’ 也可以构造一个password=’’ or ‘1’='1’

那么其实只要在用户名与密码处都填写'or '1'='1 就可以了

[![image](/image/2015/02/image_thumb13.png "image")](/image/2015/02/image13.png) 

&#160;

或者在用户名里输入admin'or '1'='1 密码不输入，这个是在猜测其有一个叫admin的用户