title: Android官方的uiautomator例子的实现
id: 1264
permalink: 1264
categories:
  - 计算机相关
date: 2014-12-29 01:04:49
tags:
  - android
---
Android的自动化测试有很多框架，其中uiautomator是google官方提供的黑盒UI相关的自动化测试工具，case使用java写，今天实践了一下[官方文档](http://developer.android.com/tools/help/uiautomator/UiDevice.html)中样例程序，其中还是有一些小问题需要总结一下的。
前几天试着使用uiautoamtor在真实的项目中写了一个简单的测试
[使用Uiautomator做基于UI界面的测试](http://www.yangyanxing.com/article/use-uiautomator-for-uitest.html)

# 使用ADT创建一个java的项目

    在创建项目的时候要加上JUnit与你使用的Android platforms中对应的android.jar与uiautomator.jar

[![image](/image/2014/12/image16.png "image")](/image/2014/12/image16.png) 
<!-- more -->
# 新建一个包(我这里就只叫com)

# 再这个包下创建一个class
输入以下java代码,代码全是官方文档上的代码，除了最上面的package

 ```  java
package com;

import com.android.uiautomator.core.UiObject;
import com.android.uiautomator.core.UiObjectNotFoundException;
import com.android.uiautomator.core.UiScrollable;
import com.android.uiautomator.core.UiSelector;
import com.android.uiautomator.testrunner.UiAutomatorTestCase;

public class Runer extends UiAutomatorTestCase {

   public void testDemo() throws UiObjectNotFoundException {

      // Simulate a short press on the HOME button.
      getUiDevice().pressHome();

      // We’re now in the home screen. Next, we want to simulate
      // a user bringing up the All Apps screen.
      // If you use the uiautomatorviewer tool to capture a snapshot
      // of the Home screen, notice that the All Apps button’s
      // content-description property has the value “Apps”.  We can
      // use this property to create a UiSelector to find the button.
      UiObject allAppsButton = new UiObject(new UiSelector()
         .description("Apps"));

      // Simulate a click to bring up the All Apps screen.
      allAppsButton.clickAndWaitForNewWindow();

      // In the All Apps screen, the Settings app is located in
      // the Apps tab. To simulate the user bringing up the Apps tab,
      // we create a UiSelector to find a tab with the text
      // label “Apps”.
      UiObject appsTab = new UiObject(new UiSelector()
         .text("Apps"));

      // Simulate a click to enter the Apps tab.
      appsTab.click();

      // Next, in the apps tabs, we can simulate a user swiping until
      // they come to the Settings app icon.  Since the container view
      // is scrollable, we can use a UiScrollable object.
      UiScrollable appViews = new UiScrollable(new UiSelector()
         .scrollable(true));

      // Set the swiping mode to horizontal (the default is vertical)
      appViews.setAsHorizontalList();

      // Create a UiSelector to find the Settings app and simulate
      // a user click to launch the app.
      UiObject settingsApp = appViews.getChildByText(new UiSelector()
         .className(android.widget.TextView.class.getName()),
         "Settings");
      settingsApp.clickAndWaitForNewWindow();

      // Validate that the package name is the expected one
      UiObject settingsValidation = new UiObject(new UiSelector()
         .packageName("com.android.settings"));
      assertTrue("Unable to detect Settings",
         settingsValidation.exists());
      UiObject reportBug = new UiObject(new UiSelector().text("Sound"));
      reportBug.clickAndWaitForNewWindow();
      UiObject soundValidation = new UiObject(new UiSelector()
      .text("Volumes"));
   assertTrue("Unable to detect Sound",
   soundValidation.exists());

      getUiDevice().pressHome();

  }
```

# 使用ant工具生成build.xml

我这里在使用ADT自已的ant插件时提示

      Class not found: javac1.8  
网上查了查，是插件与我java环境不符，下载最新的ant插件就可以了
[http://ant.apache.org/bindownload.cgi](http://ant.apache.org/bindownload.cgi "http://ant.apache.org/bindownload.cgi") 
[![image](/image/2014/12/image17.png "image")](/image/2014/12/image17.png)
下载这个tar.gz包，解压，然后将apache-ant-1.9.4bin目录添加到环境变量PATH中
然后cmd到android sdk的tools目录，使用andrlid list命令，记住你将要在模拟器中运行的(也是你刚刚导入android.jar与uiautomator.jar包时所在的platforms)
[![image](/image/2014/12/image18.png "image")](/image/2014/12/image18.png) 
在cmd下使用android create uitest-project -n <name> -t <android-sdk-ID> -p <path>
-n 为生成的jar包名称，自已任意定义，
-t 为上面查看到的值，我这里是1
-p 为输出路径，这里就是刚才创建的java项目所在的路径
android create uitest-project -n AutoRunner -t 1 -p D:myAndroidStudyandroidTest
然后再cmd进入D:myAndroidStudyandroidTest，使用ant build命令生成AutoRunner.jar文件
# 将这个AutoRunner.jar文件push到模拟器中
    adb push AutoRunner.jar /data/local/tmp
# 使用adb shell uiautomator runtest AutoRunner.jar –c com.Runer 使Runer类运行
[![image](/image/2014/12/image19.png "image")](/image/2014/12/image19.png) 
我的代码里又在官方基础上多了一个点击”sound”的操作与点击Home键操作
```  java
UiObject reportBug = new UiObject(new UiSelector().text("Sound"));
      reportBug.clickAndWaitForNewWindow();
      UiObject soundValidation = new UiObject(new UiSelector()
      .text("Volumes"));
   assertTrue("Unable to detect Sound",
   soundValidation.exists());

```
