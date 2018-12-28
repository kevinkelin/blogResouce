title: 利用新浪SAE storage服务做图床
id: 460
permalink: 460
categories:
  - PHP
date: 2012-12-10 23:37:16
tags:
---

新浪SAE 为用户提供storage服务，来永久性存储文件，这很好的为用户提供图片等文件存储服务，以下通过方法来建立一个自己的图床

需求：上传一张照片，返回这张照片的地址

实现 http://kevinkelin.sinaapp.com/fileupload.php

# 首先在SAE上建立一个storage 的domain
首先在SAE上建立一个storage 的domain,(其实再之前你需要有一个应用，这里不写了)
 <!-- more -->
# 新建一个fileupload.php文件
代码如下
``` php
<?php
session_start();
include_once('saestorage.class.php');

?>

<html>

<body>
<form action="fileupload.php" method="post" enctype="multipart/form-data">
<input type = "file" name="myfile" size="100" /><br>
<input type = "submit" value= "upload" / >
</form>
</body>
</html>

<?php
$domain="imagefile";
$file_name = $_FILES["myfile"]["name"];
$temp_arr = explode(".", $file_name);
$file_ext = array_pop($temp_arr);
$file_ext = trim($file_ext);
$file_ext = strtolower($file_ext);
$new_file_name = date("YmdHis") . '_' . rand(10000, 99999).'.'.$file_ext;
$s = new SaeStorage();
//$s->upload( 'imagefile',$_FILES["myfile"]["name"],$_FILES["myfile"]["name"]);
echo $aimage=$s->upload( 'imagefile',$new_file_name,$_FILES["myfile"]["tmp_name"]);

?>
```

得到了图片的地址，你就可以随意的使用了，前提是你在建立domain的时候不要防盗链，新浪SAE也是有限制的
运行在SAE上的应用(App)将会消耗平台资源，为保证各App不互相 影响，我们引入了*分钟配额*的概念，即：在每分钟内每个应用的各个服务所消耗的 资源的速度。当[Storage](http://sae.sina.com.cn/?m=devcenter&catId=204)服务超过分钟配额，我们将会立即禁掉该应用的[Storage](http://sae.sina.com.cn/?m=devcenter&catId=204) 服务，禁用五分钟后，恢复会自动恢复，避免影响到SAE平台的稳定性。服务因为超过“分钟配额”而被禁用时，会在“服务状态”看到该服务被禁用的原因是：**OverMinuteQuota   **
服务请求数cpu时间流入带宽流出带宽
Storage5,000NA80MB400MB
今天又将这个页面改了改，实现可以遍历指定domain下所有图片以及下载地址，并且实现后上传先显示，只是实现功能，所以页面太糙了。。。
``` php
<?php
session_start();

include_once( 'config.php' );
include_once( 'saetv2.ex.class.php' );
include_once('saestorage.class.php');

?>

<html>

<body>
<form action="fileupload.php" method="post" enctype="multipart/form-data">
<input type = "file" name="myfile" size="100" /><br>
<input type = "submit" value= "upload" / >
</form>
</body>
</html>

<?php
$domain="imagefile";
$file_name = $_FILES["myfile"]["name"];
$temp_arr = explode(".", $file_name);
$file_ext = array_pop($temp_arr);
$file_ext = trim($file_ext);
$file_ext = strtolower($file_ext);
$new_file_name = date("YmdHis") . '_' . rand(10000, 99999).'.'.$file_ext;
$s = new SaeStorage();
//$s->upload( 'imagefile',$_FILES["myfile"]["name"],$_FILES["myfile"]["name"]);
echo $aimage=$s->upload( 'imagefile',$new_file_name,$_FILES["myfile"]["tmp_name"]);

$files = $s->getList($domain,"",100);
echo "<pre>";
print_r($files);
echo "</pre>";

foreach($files as $name){
echo $s->getUrl($domain,$name)."<br>";
}
echo "<br>";

foreach($files as $name){
  //echo "<img src=".$s->getUrl($domain,$name)." width='50%'. height='100%'.>";

  echo "<table border='5' align='center'>";
  echo "<tr>";

                echo "<td align='right'>";
  echo "<a target='_blank' href=".$s->getUrl($domain,$name). "><img src=".$s->getUrl($domain,$name)." width='30%' height='30%'"."></a>";
                        echo "<br><br>";
                        echo $s->getUrl($domain,$name)."<br>";
  echo "</td>";

  echo "</tr>";

  echo "</table>";
}

?>
```