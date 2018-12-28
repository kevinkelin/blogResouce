title: 使用jquery在一个页面中有多个form表单做数据有效性校验
tags:
  - javascript
  - jquery
id: 1381
permalink: 1381
comment: false
categories:
  - web
date: 2015-10-02 16:23:48
---

最近在做一个小网站的项目，有一个小问题，可能在有经验的前端er面前不是什么问题，但是由于我接触前端很少，所以这个问题也搞了好一会才解决

在一个页面里有两个表单，在各自点击提交时，先要对相应的input里做非空校验，然后再对里面的数据做下简单的字符串判断
 <!-- more -->
html里是这样的
  ```  html
<tbody>
    <tr>
        <td class="sync"><a class="btn btn-success"  href="/syncfile?redo=1&systype=x86">32位同步其他组文件</a></td>
        <td class="update"><a class="btn btn-success"  href="/selfupdate?redo=1&systype=x86" >32位急救箱自升级文件</a></td>
        <td class="twice">
            <form class="form-inline" action="/check2" method="GET">
              <input type="hidden" name="type" value="zip" />
              <input type="hidden" name="redo" value=1 />
              <input type="text" class="form-control sftppath" name="sftppath" placeholder="请输入急救箱捆包地址">
              <button type="submit" class="btn btn-primary">急救箱捆包二次</button>
            </form>
        </td>
    </tr>
    <tr>
        <td class="sync"><a class="btn btn-success"  href="/syncfile?redo=1&systype=x64">64位同步其他组文件</a></td>
        <td class="update"><a class="btn btn-success"  href="/selfupdate?redo=1&systype=x64" >64位急救箱自升级文件</a></td>
        <td class="twice">
            <form class="form-inline" action="/check2" method="GET">
              <input type="hidden" name="type" value="update" />
              <input type="hidden" name="redo" value=1 />
              <input type="text" class="form-control sftppath" name="sftppath" placeholder="请输入急救箱升级地址">
              <button type="submit" class="btn btn-primary" id="123">急救箱升级二次</button>
            </form>
        </td>
    </tr>
    </tbody>
```

&#160;

起初我只是简单的用jquery做下非空校验

```  javascript
$("form").submit(function(){
        if($("input.sftppath").val()==''){
            alert("sftp不能为空");
        }

```

这样写，在点击第一个按钮(急救箱捆包二次)时，当前面的input里没有写数据的时候会有alert提示，

但是点击第二个按钮(急救箱升级二次)时，即使在前面的input里填写上数据也会有alert提示，因为在急救箱捆包二次那的input里没有值。

于是在网上找了一下，没有使用第三方的jquery插件，因为这个功能太小了。

我先打印了一个$(this).html()

$("form").submit(function(){

alert($(this).html())

})


在submit里，$(this)所指的节点是form的根节点

于是这里就好弄了

使用$(this).children("input.sftppath").val()来获得相应form中的sftppath的内容

```  javascript
$("form").submit(function(){
    if($(this).children("input.sftppath").val() == ''){
        alert("sftp路径不能为空");
        return false;
    }else{
        var buttontext = $(this).children('button.btn-primary').text();
        var sftppath = $(this).children("input.sftppath").val();
        if (buttontext == "急救箱升级二次"){
            if(sftppath.indexOf('-all')==-1){
                alert("不是有效的升级sftp路径");
                return false;
            }else{
                return true;
            };
        }else if (buttontext == "急救箱捆包二次"){
            if(sftppath.indexOf('-zip') == -1){
                alert("不是有效的捆包sftp路径");
                return false;
            }else{
                return true;
            };
        };
    };

});

```

其实这里也就是明白了在submit函数里，$(this)指的是form的根节点，而不是那个button节点。