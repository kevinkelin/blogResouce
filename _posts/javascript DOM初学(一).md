title: javascript DOM初学(一)
id: 994
permalink: 994
categories:
  - web
date: 2014-02-23 02:18:25
tags:
  - javascript
---

开始学习javacript，这里做个总结
<!-- more -->
``` html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>无标题文档</title>
<style>
#div1 {width:200px;height:200px;background:#CCC;position:relative}
#div2 {width:100px;height:100px;background:red;position:absolute;left:50px;top:50px;}
</style>
<script>
window.onload=function()
{
    var oul = document.getElementById('ul1')
    //alert (oUL.childNodes.length)
    for(var i=0;i<oul.childNodes.length;i++) //childNodes属性随着浏览器的不同而不同，高级点的回车换行等也算一个
    {
        if (oul.childNodes[i].nodeType == 1) //这里就解决了只选用元素属性节点，文本属性的节点就不算了，也就排除了回车换行等问题
        {
            oul.childNodes[i].style.background = 'red'
        }
    }
 
    for(var i=0;i<oul.children.length;i++)
    {
        oul.children[i].style.background = 'blue' //children属性直接只选取元素属性，不算文本元素
    }
 
    var oa = document.getElementsByTagName('a')
    for(var i = 0;i<oa.length;i++)
    {
        oa[i].onclick=function() //为某个元素添加某些方法
        {
            //获取父节点
            this.parentNode.style.display='none'    //使得父节点设置为隐藏
        }
    }
 
    var od=document.getElementById('div2')
    od.onclick=function()
    {
        alert(od.offsetParent)//获取该元素用于定位的元素
    }
 
    var oul3=document.getElementById("ul3")
    //oul3.firstElementChild.style.background = 'red'
    if (oul3.firstElementChild)//处理兼容问题
    {
        oul3.firstElementChild.style.background = 'blue'
    }
    else
    {
        oul3.firstChild.style.background = 'red'
    }
 
    var osub = document.getElementById('sub')
    var otext = document.getElementById('text')
    var ouser = document.getElementById("userType")
    var yyx = document.getElementsByTagName('p')[0]//返回是列表，所以得取其中之一
    osub.onclick = function()
    {
        //1.otext.value = 'hahaha'
        //2.otext['value'] = 'hahaha'
        var text = ouser.getAttribute('value')
        var text2 = yyx.firstChild.nodeValue //获得该节点的值，也就是文本元素
        otext.setAttribute('value',text2)
    }
 
    function getClassName(parent,classname)
    {
        classnamelist = []
        var d = document.getElementById(parent)
        var c = d.getElementsByTagName('*')
        for (var i=0;i<c.length;i++)
        {
            if (c[i].className == classname)
            {
                classnamelist.push(c[i])
            }
        }
        return classnamelist
    }
 
    var oul4 = document.getElementById("ul4")
    var oli4 = oul4.getElementsByTagName('li')
    //var dli4 = document.getElementsByTagName('li') 会选取整个文档内的元素，并不是我们想要的，我们想要的是从某个父级元素下面选元素
 
    needRed = getClassName('ul4','red')
    for(var i = 0;i<needRed.length;i++)
    {
        needRed[i].style.background = 'red'
    }
 
};
</script>
</head>
 
<body>
<ul id="ul1">
    <li></li>
    <li></li>
    <li></li>
    <li></li>
</ul>
<br>
<ul id="ul2">
    <li>adfadf <a href="#">隐藏</a></li>
    <li>aadfaddf <a href="#">隐藏</a></li>
    <li>233df <a href="#">隐藏</a></li>
    <li>fafadf <a href="#">隐藏</a></li>
</ul>
 
<br>
 
<div id="div1">
    <div id="div2">sdasdfafasd</div>
</div>
 
<ul id="ul3">
    <li>1</li>
    <li>2</li>
    <li>3</li>
    <li>4</li>
</ul>
<br />
<p>我是杨彦星</p>
<input type="text" id="userType" value="adfadfadf"/>
<input type="text" id="text" />
<input type="submit" id="sub" value="点击" />
 
<br />
 
<ul id="ul4">
    <li class="red"></li>
    <li></li>
    <li class="red"></li>
    <li></li>
    <li class="red"></li>
</ul>
 
</body>
</html>

```
