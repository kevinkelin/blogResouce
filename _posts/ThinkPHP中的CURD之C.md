title: ThinkPHP中的CURD之C
id: 373
permalink: 373
categories:
  - PHP
date: 2012-08-10 01:53:09
tags:
---

CURD 增 删 改 查

C

create方法

每当实例化一个数据对象后，当需要接收post来值的时候，要用create方法来创建一个对象，根据表单提交的POST数据创建数据对象，并保存在内存中，可以通过dump($user)查看，返回值是一个数组
``` php
$user=M('user');

$vo=$user->create();

dupm($vo); //这时会遍历出得到的（post）值
```

 <!-- more -->
<pre >array(2) {
  ["username"] => string(4) "root"
  ["password"] => string(11) "12345678"
}</pre>
 

写一个add方法，加上一个表单

在tpl里面建立一个表单
``` php
<html>
<body>

<form action="__URL__/add" method="post">
用户名<input type="text" name="username"><br>
密     码<input type="password" name="password"><br>
<input type="submit" value="提交">

</form>

</body>
</html>
```

在user的控制器里面写上一个方法，这时也可以不写create方法，但是就是单独写上一个对象的赋值方法，create的时候其实就是在接收post而来的数据
``` php
function add(){
$user=M('user');
//$user->create(); 如果不用create方法，也可以用下面的方法
$user->username=$_POST['username'];
$user->password=md5($_POST[password]);//将post来的密码进行md5加密
                        /*
                        如果用create方法可以用这面的方法写
                        $user->create();
                        $user->password=md5($user->password);
                        */
                        //add为thinkphp中向数据库中添加数据的方法

if ($user->add()) {
$this->success('添加用户成功');
}else {
$this->error('添加用户失败');
}

}
```