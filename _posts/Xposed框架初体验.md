title: Xposed框架初体验
comment: true
categories:
  - Android
date: 2016-06-12 00:05:17
tags: 
  - android
permalink: first-use-Xposed

---

想必很多人都听说过微信抢红包插件，但是很少有人想过它是怎么实现的，以前我以为是可能通过监听某个消息广播或者什么的，但是前几天在testerhome中看到有一篇介绍Xposed框架的文章[用黑客思维做测试——神器 Xposed 框架介绍](https://testerhome.com/topics/3819),我觉得这应该是广大抢红包插件的实现。正好有个同事和我说过有一个微信计步的作弊器(汗，怎么这个东西净用在这方面呢)[手把手教你当微信运动第一名](http://drops.wooyun.org/tips/8416),于是对这个大名鼎鼎的Xposed学习了一番，觉得它有很多潜能！
<!-- more -->

参考文章
[用黑客思维做测试——神器 Xposed 框架介绍](https://testerhome.com/topics/3819)
[Android Hook神器：XPosed入门与登陆劫持演示](http://www.csdn.net/article/2015-08-14/2825462)
[Xposed Home](http://repo.xposed.info/)

# 关于Xposed

Xposed 框架是一款可以在不修改APK的情况下影响程序运行（修改系统）的框架服务，基于它可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运作。

# 基本原理

Zygote 进程是 Android 的核心，所有的应用程序进程以及系统服务进程都是由Zygote进程 fork 出来的。Xposed Framework 深入到了 Android 核心机制中，通过改造 Zygote 来实现一些很牛逼的功能。Zygote 的启动配置在/init.rc 脚本中，由系统启动的时候开启此进程，对应的执行文件是/system/bin/app_process，这个文件完成类库加载及一些初始化函数调用的工作。 当系统中安装了 Xposed Framework 之后，会拿自己实现的 app_process 覆盖掉 Android 原生提供的文件，使得app_process在启动过程中会加载XposedBridge.jar这个jar包，从而完成对Zygote进程及其创建的Dalvik虚拟机的劫持。


# 安装

在android5.0以上要使用不同的包，具体可以参考http://repo.xposed.info/module/de.robv.android.xposed.installer
为了方便我使用了一台android4.4的手机，只要安装一个apk即可，手机要有root，因为安装Xposed的时候需要root，但是一旦安装成功则不再需要root了，安装后启动
![Xposed](/image/xposed.jpg)

按照上面的说明点击安装，如果顺利的话重启后就可以正常的使用了
![Xposed安装成功](/image/XposedSuc.png)

# 写hook模块使用

Xposed网站已经有很多别人写好的模块，可以直接下载安装使用，也可以自已写模块，模块其实就是一个apk，按照一定的规则生成的，当安装模块以后，Xposed会自已识别。
开始自已写一个小demo吧，采用testerhome中的那个hook取时间的例子。
## 首先先写测试的apk

``` java
package com.example.showtimer;

import java.util.Calendar;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.Window;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_main);
        final TextView tv = (TextView) findViewById(R.id.tv);
        Button show = (Button) findViewById(R.id.showTimer);
        show.setOnClickListener(new OnClickListener() {
            public void onClick(View v) {
                Calendar c = Calendar.getInstance();
                int year = c.get(Calendar.YEAR);
                int month = c.get(Calendar.MONTH);
                int day = c.get(Calendar.DAY_OF_MONTH);
                int hour = c.get(Calendar.HOUR);
                int min = c.get(Calendar.MINUTE);
                String time = ""+year+"-"+month+"-"+day+" "+hour+":"+min;
                tv.setText(time);
                
            }
        });

}

```

这个apk很简单，点击按钮后，显示了当前的时间

![显示当前时间](/image/showTime.png)

## 写hook模块

只要修改系统的java.util.Calendar类中的get函数，并修改相应的返回值则能达到hook目的

### 新建一个hookTest的android项目并将XposedBridgeApi-54.jar加入项目的build-path中
api下载地址 http://forum.xda-developers.com/xposed/xposed-api-changelog-developer-news-t2714067
在项目中新建一个lib目录，将jar包放进去，添加到build paht中
![将xposed加入到build path中](/image/xposedapi.png)

### 新建一个Hook类实现IXposedHookLoadPackage接口
实现handleLoadPackage方法
``` java
public void handleLoadPackage(LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.example.showtimer"))  
            return;  
        XposedBridge.log("Loaded app name   : " + lpparam.packageName);
    }
```

上面的代码定义了只有packageName是com.example.showtimer才进行hook操作，其它的放过

`findAndHookMethod()` 方法是找到并且Hook方法
findAndHookMethod的参数说明
第一个参数定义了要hook的类，对于某个akp来说就是其包名+activity，第二个参数是个固定的lpparam.classLoader，第三个是要hook的方法名，之后的参数是这个方法需要的参数类型class，有几个就写几个，最后是初始化一个XC_MethodHook的对象。然后重写其beforeHookedMethod与afterHookedMethod方法。
``` java
findAndHookMethod("java.util.Calendar", lpparam.classLoader,"get",int.class,new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called before the clock was updated by the original method
                XposedBridge.log("Enter->beforeHookedMethod:Calendar.get");
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called after the clock was updated by the original method
                XposedBridge.log("Enter->afterHookedMethod:Calendar.get");
                param.setResult((int)11);
            }
        });
```

在findAndHookMethod中重写了两个方法，beforeHookedMethod和afterHookedMethod，看名字也能猜出它们的作用，before是在hook之前，可以得到一些正常的值，after那个函数则可以修改一些返回值。上面这里就是将java.util.Calendar这个类的get方法所有返回值都修改为11

Hook.java的总代码为
``` java
package com.example.xposedtest;

import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;
import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;

import java.util.ArrayList;
import java.util.List;

import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import android.widget.Toast; 


public class Hook implements IXposedHookLoadPackage {

    @Override
    public void handleLoadPackage(LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.example.showtimer"))  
            return;  
        XposedBridge.log("Loaded app name   : " + lpparam.packageName);
        findAndHookMethod("java.util.Calendar", lpparam.classLoader,"get",int.class,new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called before the clock was updated by the original method
                XposedBridge.log("Enter->beforeHookedMethod:Calendar.get");
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called after the clock was updated by the original method
                XposedBridge.log("Enter->afterHookedMethod:Calendar.get");
                param.setResult((int)11);
            }
        });
    }

}

```

### 声明主入口路径
需要在assets文件夹中新建一个xposed_init的文件，并在其中声明主入口类。如这里我们的主入口类为com.example.xposedtest.Hook

### 在AndroidManifest.xml文件中配置插件名称与Api版本号
``` xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.xposedtest"
    android:versionCode="1"
    android:versionName="4.0" >

    <uses-sdk
        android:minSdkVersion="17"
        android:targetSdkVersion="21" />

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <meta-data  
            android:name="xposedmodule"  
            android:value="true" />  
            
        <meta-data  
            android:name="xposeddescription"  
            android:value="劫持获得时间函数" />
        
        <meta-data  
            android:name="xposedminversion"  
            android:value="30" />  
            
        
        <activity
            android:name=".MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>

```

### 编译并安全这个apk
Xposed会自已识别，打开Xposed的"模块"，启用刚才安装的模块
![安装好的模块](/image/xposedmodule.png)

### 再打开之前写的showTime应用，点击按钮看看
![被hook后的时间](/image/errorTime.png)

使用XposedBridge.log() 记录的log可以在Xposed的日志里看到

# Hook自定义的函数

刚才hook的是系统的函数，下面写一个hook自定义的函数
还是在刚才showTime的项目中再加一个用户登录的模拟操作
``` java
package com.example.showtimer;

import java.util.Calendar;

import android.app.Activity;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.Window;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_main);
        final EditText usename = (EditText) findViewById(R.id.username);
        final EditText password = (EditText) findViewById(R.id.password);
        Button login = (Button) findViewById(R.id.button1);
        final TextView tv = (TextView) findViewById(R.id.tv);
        Button show = (Button) findViewById(R.id.showTimer);
        show.setOnClickListener(new OnClickListener() {
            public void onClick(View v) {
                Calendar c = Calendar.getInstance();
                int year = c.get(Calendar.YEAR);
                int month = c.get(Calendar.MONTH);
                int day = c.get(Calendar.DAY_OF_MONTH);
                int hour = c.get(Calendar.HOUR);
                int min = c.get(Calendar.MINUTE);
                String time = ""+year+"-"+month+"-"+day+" "+hour+":"+min;
                tv.setText(time);
                
            }
        });
        
        login.setOnClickListener(new OnClickListener() {
            public void onClick(View v) {
                String user = usename.getText()+"";
                String pass = password.getText()+"";
                if (validate(user,pass)) {
                    Toast.makeText(MainActivity.this, "登录成功", Toast.LENGTH_LONG).show();
                }else{
                    Toast.makeText(MainActivity.this, "登录失败", Toast.LENGTH_LONG).show();
                }               
            }
        });
        
        
    }

    private boolean validate(String user, String pass) {
            if (user.equals("yang")&&pass.equals("123")) {
                return true;
            }
            return false;
    }

}

```

这里自定义了一个validate方法，校验user为"yang"且pass为123则返回真，Toast显示登录成功
其它的都返回假。

接着再刚才的Hook类中写hook方法
``` java
package com.example.xposedtest;

import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;
import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;

import java.util.ArrayList;
import java.util.List;

import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import android.widget.Toast; 


public class Hook implements IXposedHookLoadPackage {

    @Override
    public void handleLoadPackage(LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.example.showtimer"))  
            return;  
        XposedBridge.log("Loaded app name   : " + lpparam.packageName);
        XposedBridge.log("Loaded app process: " + lpparam.processName);
        XposedBridge.log("Loaded app appInfo: " + lpparam.appInfo);
        findAndHookMethod("java.util.Calendar", lpparam.classLoader,"get",int.class,new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called before the clock was updated by the original method
                XposedBridge.log("Enter->beforeHookedMethod:Calendar.get");
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called after the clock was updated by the original method
                XposedBridge.log("Enter->afterHookedMethod:Calendar.get");
                param.setResult((int)11);
            }
        });
        
        findAndHookMethod("com.example.showtimer.MainActivity", lpparam.classLoader, "validate", String.class,String.class,new XC_MethodHook(){
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                XposedBridge.log("Enter->beforeHookedMethod:validate");
                XposedBridge.log("afterHookedMethod userName:" + param.args[0]);   
                //传入参数2   
                XposedBridge.log("afterHookedMethod pass:" + param.args[1]);  
                //函数返回值  
                XposedBridge.log("afterHookedMethod result:" + param.getResult());
            }
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called after the clock was updated by the original method
                XposedBridge.log("Enter->afterHookedMethod:validate");
                param.setResult(true);
                XposedBridge.log("afterHookedMethod userName:" + param.args[0]);   
                //传入参数2   
                XposedBridge.log("afterHookedMethod pass:" + param.args[1]);  
                //函数返回值  
                XposedBridge.log("afterHookedMethod result:" + param.getResult());  
            }
            
        });
}

```

在afterHookedMethod方法中将param.setResult(true);则无论输入什么或者不输入都会返回真，这样无论如何都能显示一个Toast登录成功。

![登录成功](/image/loginSuc.png)

后记：
如果测试的apk用了混淆，则hook的时候也要修改相应的函数为混淆后的函数。
Xposed这个工具可以做很多事情，在测试行业里可以用于构造一些很不好模拟的环境，另外各种插件也能玩的很6。