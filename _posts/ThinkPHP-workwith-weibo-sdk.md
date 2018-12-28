title: 在ThinkPHP整合新浪微博SDK
comment: true
categories:
  - PHP
date: 2016-02-10 22:28:32
tags: 
  - ThinkPHP
  - PHP
  - 微博
permalink: ThinkPHP-workwith-weibo-sdk

---

最近在玩PHP，在看了基本语法以后就开始看ThinkPHP了，几年以前接触过一些，但是基本上都忘的差不多了
现在再看的时候，发现版本更新了好多，添加了很多新的功能特性，使用时候也有很多不一样的，之前写了一篇边看边写的笔记，[ThinkPHP的学习笔记](http://www.yangyanxing.com/article/thinkphp-study-note.html) 
今天结合[官方的文档](http://document.thinkphp.cn/manual_3_2.html)来整合一下新浪微博的SDK，在使用的过程中还是有一些问题需要注意的，我使用的是3.2.3版本
<!-- more -->

# ThinkPHP和项目与模块的初始化

这个就不详细写了，学习笔记中有记录，也是比较基础的

# 整合新浪微博的SDK

下载SDK [新浪微博SDK](http://open.weibo.com/wiki/SDK#PHP_SDK)
下载后将压缩包中的saetv2.ex.class.php取下来，重新命名，叫什么无所谓，只是为了去掉那个点，
我重新命名为saetv2.class.php, `.class.php` 是必须要保留的，我将其放到了ThinkPHP/Library/Org/Com/Sina/ 目录下，文档上说这个目录下的类库是可以自动加载的，但是文档上使用的是namespace的方式
我一开始也按照文档的介绍使用namespace方式，但是后来发现这种以名称空间的方式有很多限制，首先一个文件里只能有一个类且文件名还要和这个类名对应，微博SDK里有三个类，如果按照namespace的方法，我还得新建一个文件重新拷贝一下相应的类，很麻烦，后来我直接采用import这个类的方式，使用起来方便了很多。

# 在WeiboController.class.php中写逻辑

首先把新浪微博sdk引入进来，在use Think\Controller; 下面加入 引用
> import('Org\Com.Sina\saetv2');

``` php
<?php
namespace Home\Controller;
use Think\Controller;
import('Org\Com.Sina\saetv2');

```

这里的目录结构中的`com\sina` 要改成`Com.Sina`
下面如果要使用saetv2中的类就可以直接使用 \类名调用，如使用SaeTOAuthV2类就可以使用
`$c = new \SaeTClientV2()`

# 获得token

其实里面的代码基本上就是sdk中的callback.php中的代码,这里又加上了ThinkPHP对于session与cookie的封装

``` php
$o = new \SaeTOAuthV2( WB_AKEY , WB_SKEY );
$code_url = $o->getAuthorizeURL( WB_CALLBACK_URL );
if(isset($_GET['code'])){
    $code = $_GET['code'];
    $keys = array();
    $keys['code'] = $_REQUEST['code'];
    $keys['redirect_uri'] = WB_CALLBACK_URL;
    $token = $o->getAccessToken( 'code', $keys ) ;
    if (!empty($token['uid']) && !empty($token['access_token'])) {
       session('token',$token);
       cookie('weibojs_'.$o->client_id, http_build_query($token));
       $this->success('新浪微博授权成功',U('index','',''),3);
    }
    else{
       $this->error('新浪微博授权失败',U('index','',''),3); 
    }
```

# 调用微博的接口

当获得了token后就可以使用token访问新浪微博的接口了，[接口文档](http://open.weibo.com/wiki/%E5%BE%AE%E5%8D%9AAPI)

全部代码如下
``` php
<?php
namespace Home\Controller;
use Think\Controller;
import('Org\Com.Sina.weibo\saetv2');
class WeiboController extends Controller {    
    public function index(){
        // session('[destroy]');
        define('WB_AKEY', C('AKEY'));
        define('WB_SKEY', C('SKEY'));
        define('WB_CALLBACK_URL', C('CALLBACK'));

        if (session('token')) {
            dump(session('token'));
            $c = new \SaeTClientV2( WB_AKEY , WB_SKEY , $_SESSION['token']['access_token'] );
            // 这个是调用发微博的接口
            // $up = $c->update("使用ThinkPHP发微博".(string)(mt_rand()));
            $ht = $c->home_timeline();
            $userinfo = $c->show_user_by_id(session('token')['uid']);
            dump($userinfo);
            echo "点此<a href=".U('logout','','').">退出登录</a>";
        }else{
            // 这个是使用namespace方式调用，麻烦
            // $o = new \Org\Com\Sina\SaeTOAuthV2( WB_AKEY , WB_SKEY );
            // 
            // 下面这个是使用import库以后使用方法，比较简单
            $o = new \SaeTOAuthV2( WB_AKEY , WB_SKEY );
            $code_url = $o->getAuthorizeURL( WB_CALLBACK_URL );
            if(isset($_GET['code'])){
                $code = $_GET['code'];
                $keys = array();
                $keys['code'] = $_REQUEST['code'];
                $keys['redirect_uri'] = WB_CALLBACK_URL;
                // dump($keys);
                $token = $o->getAccessToken( 'code', $keys ) ;
                // dump($token);
                if (!empty($token['uid']) && !empty($token['access_token'])) {
                   session('token',$token);
                   cookie('weibojs_'.$o->client_id, http_build_query($token));
                   $this->success('新浪微博授权成功',U('index','',''),3);
                }
                else{
                   $this->error('新浪微博授权失败',U('index','',''),3); 
                }

            }
            echo "您还没有登录，请使用<a href=".$code_url.">新浪微博</a>进行登录";
        }
        $this->show('<style type="text/css">*{ padding: 0; margin: 0; } div{ padding: 4px 48px;} body{ background: #fff; font-family: "微软雅黑"; color: #333;font-size:24px} h1{ font-size: 100px; font-weight: normal; margin-bottom: 12px; } p{ line-height: 1.8em; font-size: 36px } a,a:hover{color:blue;}</style><div style="padding: 24px 48px;"> ','utf-8');

    }


    public function logout(){
        // 退出实际上就是简单地调用了销毁session的方式
        session('[destroy]');
        cookie(NULL);
        $this->success('您已经退出登录',U('index','',''),2);
    }
}
```

当访问 index.php/Weibo/index 时，显示授权链接
![点击授权](/image/20160210231742.png
)

点击后使用微博账号进行登录，成功后跳转回index.php/Weibo/index 

![dump_usre_info](/image/20160210232553.png
)




