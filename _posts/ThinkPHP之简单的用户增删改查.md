title: ThinkPHP之简单的用户增删改查
id: 702
permalink: 702
categories:
  - PHP
date: 2012-08-20 00:05:15
tags:
  - ThinkPHP
---

此篇文章简单的应用了之前所介绍的CURD

在IndexAction.class.php中写入以下代码
<!-- more -->

``` php
<?php
class IndexAction extends Action{
    /*主页显示*/
    public function index(){
    $user=M('user');
    $list=$user->field(array('id','username','createip'))->select();//查出id username,createip这三个字段
    $this->assign('title','操作练习');//将操作练习分配给title
    $this->assign('alist',$list); //将上面所得到的$list分配给alist
    $this->display(); //这里需要在TPL中创建一个index.html的页面

    }

    /*删除用户操作*/
    function del(){
    $user=M('user');
    if ($user->delete($_GET['id'])){//这里采用的是GET传值
    $this->success('删除用户成功');
    }else{
    $this->error('删除用户失败');
    }

    }
    /*添加用户操作*/
    function add(){
    Load('extend');//这里是为了要应用get_client_ip()方法
    if ($_POST['password']!=$_POST['repassword']){
    $this->error('两次输入的密码不一致');

    }

    $user=D('user');//使用D方法实例化模型则要在model中创建相应的UserModel.class.php，里面可以什么方法都不写
    if ($vo=$user->create()){
    $user->createtime=time();
    $user->createip=get_client_ip();
    $user->password=md5($user->password);//将数据压入到$user变量中
    if ($user->add()){
    $this->success('用户注册成功');
    }else {
    $this->error('用户注册失败');
    }
    }else {
    $this->error($user-getError());

    }

    }
    /*显示用户修改页面，这里不是修改方法*/
    function edit(){
    $id=$_GET['id'];
    $user=M('user');
    $list=$user->where("id=$id")->find();
    $this->assign(title,'用户编辑页面');
    $this->assign('list',$list);
    $this->display();//这在后面还要建立一个edit.html的模板文件

    }
    /*这里是将用户的数据更新到数据库中* 更新的方法是save而不是update*/
    function update(){
    $user=D('user');

    if ($user->create()){

    $user->password=md5($user->password);
    if ($user->save()){
    $this->success('用户更新成功，此次更新的ID为'.$user->id);
    }else{
    $this->error($user->getError());
    }
    }else {
    $this->error($user-getError());
    }

    }
}
?>
```
还要建立两个模板文件index.html和edit.html文件

index.html
``` php
<html>
<head>
<title>{$title}</title> <!-- 这里是收到的标题变量 -->
</head>

<body>
<form action="__URL__/add" method="post" >
用户名：<input type="text" name="username"><br>
密  码：<input type="password" name="password"><br>
重复密码：<input type="password" name="repassword"><br>
<input type="submit" value="提交"><br>
</form>

<volist name='alist' id='vo'>

<li><span>ID{$vo['id']}</span> <span>用户名{$vo['username']}</span> <span>注册IP{$vo['createip']}</span> <a href='__URL__/del/id/{$vo['id']}'>删除</a> <a href='__URL__/edit/id/{$vo['id']}'>编辑</a></li>

</volist>
</body>

</html>
```
edit.html
``` php
<html>
<head>
<title> {$title}</title>
</head>
<body>
<form action="__URL__/update" method="post">
用户名：<input type="text" name="username" value="{$list['username']}"><br>
密    码：<input type="text" name="password" value="{$list['password']}"><br>
注册时间：<input type="text" name="createtime" value="{$list['createtime']}"><br>
注册IP：<input type="text" name="createip" value="{$list['createip']}"><br>
<input type="hidden" value="{$list['id']}" name='id'> <!--这里是一个隐藏表单，用于传递修改的是哪个ID -->
<input type="submit" value="修改">
<a href="__URL__">返回</a>
</form>
</body>
</html>
```