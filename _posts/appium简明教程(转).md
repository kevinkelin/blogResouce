title: appium简明教程(转)
id: 1266
permalink: 1266
donate: false
categories:
  - Python
date: 2015-01-03 20:03:13
tags:
  - 自动化测试
---

### [乙醇](http://www.cnblogs.com/nbkhic/)的自动化教程写的挺好的，以下是转自他的cnblogs上面的博客

### [appium简明教程（1）——appium和它的哲学世界](http://www.cnblogs.com/nbkhic/p/3803804.html)

### 什么是appium？

下面这段介绍来自于appium的官网。
> Appium is an open-source tool you can use to automate mobile native, mobile web, and mobile hybrid applications on iOS and Android platforms. “Mobile native apps” are those written using the iOS or Android SDKs. “Mobile web apps” are web apps accessed using a mobile browser (Appium supports Safari on iOS and Chrome on Android). “Mobile hybrid apps” have a native wrapper around a “webview” – a native control that enables interaction with web content. Projects like [Phonegap](http://phonegap.com/), for example, make it easy to build apps using web technologies that are then bundled into a native wrapper – these are hybrid apps.
> 
> Importantly, Appium is “cross-platform”: it allows you to write tests against multiple platforms (iOS, Android), using the same API. This enables a large or total amount of code reuse between iOS and Android testsuites.
我们可以从上面的介绍里获得这样的一些信息：

*   1，appium是开源的移动端自动化测试框架；
*   2，appium可以测试原生的、混合的、以及移动端的web项目；
*   3，appium可以测试ios，android应用（当然了，还有firefox os）；
*   4，appium是跨平台的，可以用在osx，windows以及linux桌面系统上；
appium的哲学<!-- more -->
> Appium was designed to meet mobile automation needs according to a certain philosophy. The key points of this philosophy can be stated as 4 requirements:
> 
> 1.  You shouldn’t have to recompile your app or modify it in any way in order to automate it.
> 2.  You shouldn’t be locked into a specific language or framework to write and run your tests.
> 3.  A mobile automation framework shouldn’t reinvent the wheel when it comes to automation APIs.
> 4.  A mobile automation framework should be open source, in spirit and practice as well as in name!
appium的设计哲学是这样的：

*   1，不需要为了自动化而且重新编译或修改测试app；
*   2，不应该让移动端自动化测试限定在某种语言和某个具体的框架；也就是说任何人都可以使用自己最熟悉最顺手的语言以及框架来做移动端自动化测试；
*   3，不要为了移动端的自动化测试而重新发明轮子，重新写一套惊天动地的api；也就是说webdriver协议里的api已经够好了，拿来改进一下就可以了；
*   4，移动端自动化测试应该是开源的；

### appium的技术架构

*   OS: Apple’s [UIAutomation](https://developer.apple.com/library/ios/documentation/DeveloperTools/Reference/UIAutomationRef/_index.html)
*   Android 4.2+: Google’s [UiAutomator](http://developer.android.com/tools/help/uiautomator/index.html)
*   Android 2.3+: Google’s [Instrumentation](http://developer.android.com/reference/android/app/Instrumentation.html). (Instrumentation support is provided by bundling a separate project, [Selendroid](http://selendroid.io/))

### appium的设计思想

> We meet requirement #2 by wrapping the vendor-provided frameworks in one API, the[WebDriver](http://docs.seleniumhq.org/projects/webdriver/) API. WebDriver (aka “Selenium WebDriver”) specifies a client-server protocol (known as the [JSON Wire Protocol](https://code.google.com/p/selenium/wiki/JsonWireProtocol)). Given this client-server architecture, a client written in any language can be used to send the appropriate HTTP requests to the server. There are already clients written in every popular programming language. This also means that you’re free to use whatever test runner and test framework you want; the client libraries are simply HTTP clients and can be mixed into your code any way you please. In other words, Appium & WebDriver clients are not technically “test frameworks” – they are “automation libraries”. You can manage your test environment any way you like!
> 
> We meet requirement #3 in the same way: WebDriver has become the de facto standard for automating web browsers, and is a [W3C Working Draft](https://dvcs.w3.org/hg/webdriver/raw-file/tip/webdriver-spec.html). Why do something totally different for mobile? Instead we have [extended the protocol](https://code.google.com/p/selenium/source/browse/spec-draft.md?repo=mobile) with extra API methods useful for mobile automation.
> 
> It should be obvious that requirement #4 is a given – you’re reading this because [Appium is open source](https://github.com/appium/appium).
首先，为了能够实现哲学里描述的第2条，也就是**不应该让移动端自动化测试限定在某种语言和某个具体的框架；也就是说任何人都可以使用自己最熟悉最顺手的语言以及框架来做移动端自动化测试**；appium选择了client-server的设计模式。只要client能够发送http请求给server，那么的话client用什么语言来实现都是可以的，这就是appium及webdriver如何做到支持多语言的；

其次，为了能够实现**不要为了移动端的自动化测试而重新发明轮子，重新写一套惊天动地的api；也就是说webdriver协议里的api已经够好了，拿来改进一下就可以了；**这个思想，appium扩展了webdriver的协议，没有自己重新去实现一套。这样的好处是以前的webdriver api能够直接被继承过来，以前的webdriver各种语言的binding都可以拿来就用，省去了为每种语言开发一个client的工作量；

最后appium当然是开源的，这也实现了哲学思想里的最后一点。

 

### [appium简明教程（2）——appium的基本概念](http://www.cnblogs.com/nbkhic/p/3803830.html)

 

 

### **Client/Server Architecture**

appium的核心其实是一个暴露了一系列REST API的server。

这个server的功能其实很简单：监听一个端口，然后接收由client发送来的command。翻译这些command，把这些command转成移动设备可以理解的形式发送给移动设备，然后移动设备执行完这些command后把执行结果返回给appium server，appium server再把执行结果返回给client。

在这里client其实就是发起command的设备，一般来说就是我们代码执行的机器，执行appium测试代码的机器。狭义点理解，可以把client理解成是代码，这些代码可以是java/ruby/python/js的，只要它实现了webdriver标准协议就可以。

这样的设计思想带来了一些好处：

*   1，可以带来多语言的支持；
*   2，可以把server放在任意机器上，哪怕是云服务器都可以；（是的，appium和webdriver天生适合云测试）

### **Session**

session就是一个会话，在webdriver/appium，你的所有工作永远都是在session start后才可以进行的。一般来说，通过POST /session这个URL，然后传入**Desired Capabilities**就可以开启session了。

开启session后，会返回一个全局唯一的session id，以后几乎所有的请求都必须带上这个session id，因为这个seesion id代表了你所打开的浏览器或者是移动设备的模拟器。

进一步思考一下，由于session id是全局唯一，那么在同一台机器上启动多个session就变成了可能，这也就是selenium gird所依赖的具体理论根据。

本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途

**Desired Capabilities**

Desired Capabilities携带了一些配置信息。从本质上讲，这个东东是key-value形式的对象。你可以理解成是java里的map，python里的字典，ruby里的hash以及js里的json对象。实际上Desired Capabilities在传输时就是json对象。

Desired Capabilities最重要的作用是告诉server本次测试的上下文。这次是要进行浏览器测试还是移动端测试？如果是移动端测试的话是测试android还是ios，如果测试android的话那么我们要测试哪个app？ server的这些疑问Desired Capabilities都必须给予解答，否则server不买账，自然就无法完成移动app或者是浏览器的启动。

具体例子如下：
> For example, we might set the `platformName` capability to `iOS` to tell Appium that we want an iOS session, rather than an Android one. Or we might set the `safariAllowPopups`capability to `true` in order to ensure that, during a Safari automation session, we’re allowed to use JavaScript to open up new windows. See the [capabilities doc](http://appium.io/slate/en/v1.1.0/?ruby#caps.md) for the complete list of capabilities available for Appium

### **Appium Server**

这就是每次我们在命令行用appium命令打开的东西。

### **Appium Clients**

由于原生的webdriver api是为web端设计的，因此在移动端用起来会有点不伦不类。appium官方提供了一套appium client，涵盖多种语言ruby/java/python，在我看来ruby client是实现最好的。在测试的时候，一般要使用这些client库去替换原生的webdriver库。这实际上不是替换，算是client对原生webdriver进行了一些移动端的扩展，加入了一些方便的方法，比如swipe之类，appium client让我们可以更方便的写出可读性更好的测试用例。

### **Appium.app, Appium.exe**

appium server的GUI版本，前者用在osx上，后者是windows上。可视化、不需要装node，可以看app的UI结构是这个东东的卖点。

 

### [appium简明教程（3）——appium的安装windows版](http://www.cnblogs.com/nbkhic/p/3803883.html)

 

appium的哲学里有一条就是不重新发明轮子。同样，官方已经有明确的安装步骤了，因此在这里纯属搬砖。

[原文地址](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md)

感谢testerhome的辛勤翻译。

本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途

##### 限制

如果你在windows上安装appium，你没法使用预编译专用于OS X的.app文件，你也将不能测试IOS apps，因为appium依赖OS X专用的库来支持IOS测试。这意味着你只能通过在mac上来运行IOS的app测试。这点限制挺大。

#### [](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md#%E5%BC%80%E5%A7%8B%E5%AE%89%E8%A3%85)开始安装

1.  安装[nodejs](http://nodejs.org/download/) 0.8版本及以上, 通过官方的安装程序来安装。
2.  安装android的sdk包，([http://developer.android.com/sdk/index.html](http://developer.android.com/sdk/index.html)), 运行依赖sdk中的'android'工具。并确保你安装了Level17或以上的版本api。设置`ANDROID_HOME`系统变量为你的Android SDK路径，并把tools platform-tools两个目录加入到系统的Path路径里。因为这里面包含有一些执行命令
3.  安装java的JDK，并设置`JAVA_HOME` 变量为你的JDK目录。
4.  安装[Apache Ant](http://ant.apache.org/bindownload.cgi) 或者直接使用Android Windows SDK自带的ant，地址在eclipseplugins目录，你需要把这个目录加到你的系统PATH变量中
5.  安装[Apache Maven](http://maven.apache.org/download.cgi). 并且设置M2HOME和M2环境变量，把M2环境变量添加到你的系统PATH变量中。
6.  安装[Git](http://git-scm.com/download/win). 确保你安装了windows下的Git，以便可以运行常用的command命令
现在，你已经下载安装了所有的依赖，开始运行 reset.bat

##### [](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md#%E8%BF%90%E8%A1%8Cappium)运行Appium

要在windows上运行测试用例，你需要先启动Android模拟器或者连接上一个API Level17以上的android真机。 然后在命令行运行appium node .

##### [](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md#%E5%A4%87%E6%B3%A8)备注

*   你必须带上--no-reset和--full-reset标记，以用于windows上的android
*   有一个硬件加速模拟器用于android，但是它有自己的一些限制，如果你想了解更多，请参考[页面](https://github.com/appium/appium/blob/master/docs/cn/android-hax-emulator.cn.md)
*   确保在你的AVD的`config.ini`中有一个配置项为`hw.battery=yes`

##### [](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md#%E6%9C%80%E7%AE%80%E7%95%A5%E7%9A%84%E5%AE%89%E8%A3%85%E6%96%B9%E5%BC%8F)最简略的安装方式

出于对官方文档的尊重，我按照原文翻译，如下介绍我的安装心得。官方提到的一些工具，其实并不需要安装。 下面介绍我已经测试过的安装和使用过程

##### [](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md#%E5%AE%89%E8%A3%85appium)安装appium

1.  安装nodejs
2、使用npm安装appium，npm install -g appium

**注意：在某些情况下，appium安装的时候并不会把appium的路径放进系统的PATH里，这时候需要手工去加一下。**

##### [](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md#%E8%BF%90%E8%A1%8Cappium-1)运行appium

启动appium，直接运行appium 即可。

##### [](https://github.com/appium/appium/blob/master/docs/cn/running-on-windows.cn.md#%E6%9B%B4%E6%96%B0appium)更新appium

通过`npm install -g appium` 来更新appium即可

 

### [appium简明教程（4）——appium client的安装](http://www.cnblogs.com/nbkhic/p/3804592.html)

 

appium client是对webdriver原生api的一些扩展和封装。它可以帮助我们更容易的写出用例，写出更好懂的用例。

appium client是配合原生的webdriver来使用的，因此二者必须配合使用缺一不可。

从本节开始，教程的内容将涵盖3个语言，ruby/python/java。

本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途

### 安装appium client

##### ruby篇（一定要在线安装）

ruby的appium client叫做appium lib，为什么是这样就不解释了，总之是历史原因。

首先update rubygem和bundler(说老实话，真的不需要，但官方文档上这么写)
> gem update --system ;
gem update bundler

然后使用gem安装
> gem uninstall -aIx appium_lib ;(这个也不是必须的)
gem install --no-rdoc --no-ri appium_lib

##### python篇（尽量在线安装）

推荐使用pip安装
> pip install Appium-Python-Client

当然了也可以在Pipy上[下载源码安装](https://pypi.python.org/pypi/Appium-Python-Client)
> tar -xvf Appium-Python-Client-X.X.tar.gz（windows上用7zip可以解压）
cd Appium-Python-Client-X.X
python setup.py install

最后，也可以通过github安装（要git客户端）
> git clone git@github.com:appium/python-client.git
cd python-client
python setup.py install

##### java篇（在线安装）

java的话用maven安装就可以了
> <dependency>
  <groupId>io.appium</groupId>
  <artifactId>java-client</artifactId>
  <version>1.3.0</version>
</dependency>

当然了，也可以自己[下载jar包](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22io.appium%22%20AND%20a%3A%22java-client%22)，请自行选择最新版本。

 

### [appium简明教程（5）——appium client方法一览](http://www.cnblogs.com/nbkhic/p/3804611.html)

 

appium client扩展了原生的webdriver client方法

下面以java代码为例，简单过一下appium client提供的适合移动端使用的新方法

*   resetApp()
*   getAppString()
*   sendKeyEvent()
*   currentActivity()
*   pullFile()
*   pushFile()
*   pullFolder()
*   hideKeyboard()
*   runAppInBackground()
*   performTouchAction()
*   performMultiTouchAction()
*   tap()
*   swipe()
*   pinch()
*   zoom()
*   getNamedTextField()
*   isAppInstalled()
*   installApp()
*   removeApp()
*   launchApp()
*   closeApp()
*   endTestCoverage()
*   lockScreen()
*   shake()
*   complexFind()
*   scrollTo()
*   scrollToExact()
*   openNotifications()
*   Context Switching: .context(), .getContextHandles(), getContext())
新增的locator

*   findElementByAccessibilityId()
*   findElementsByAccessibilityId()
*   findElementByIosUIAutomation()
*   findElementsByIosUIAutomation()
*   findElementByAndroidUIAutomator()
*   findElementsByAndroidUIAutomator()
这些方法主要覆盖了3大类：

*   driver扩展：比如增加了resetApp等操作app的方法
*   action扩展：增加一些移动端的特有的action（怎么描述呢，相当于是移动端 特有的操作），比如swipe，shake(嗯，有了这个方法就可以让代码帮你摇一摇了)等；
*   locator扩展：增加了一些移动端专属的定位策略
本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途

下一节我们开始介绍使用appium启动android模拟器

 

### [appium简明教程（6）——启动appium及android模拟器](http://www.cnblogs.com/nbkhic/p/3804637.html)

 

一般情况下，我们都从命令行启动appium。

windows下，dos命令窗口输入
> appium

如果该命令报错，那么请重装appium
> npm install -g appium

如果安装出错，请自行更换npm源。
> npm -g --registry http://registry.cnpmjs.org  install appium

**然后请打开android的模拟器，如果没有请新建一个虚拟设备。请自行解除设备锁定（手动把屏幕解锁了），以防万一。**


下面的代码以启动android原生的计算器程序为例

### ruby篇
``` ruby
require 'appium_lib'

caps   = { caps:{ platformName: 'Android', appActivity: '.Calculator', appPackage: 'com.android.calculator2' },
           appium_lib: { sauce_username: nil, sauce_access_key: nil } }
driver = Appium::Driver.new(caps).start_driver
```

讨论：可以看出ruby lib里面的Appium::Driver类实际上就是原生的webdriver类的子类，当然了，由于ruby语法灵活，也可以使用monkey patch来实现类似功能。

### python篇

``` python 
from appium import webdriver
desired_caps = {}
desired_caps['platformName'] = 'Android'
desired_caps['platformVersion'] = '4.2'
desired_caps['deviceName'] = 'Android Emulator'
desired_caps['appPackage'] = 'com.android.calculator2'
desired_caps['appActivity'] = '.Calculator'

driver = webdriver.Remote('http://localhost:4723/wd/hub', desired_caps)
```
 

讨论：webdriver.Remote实际上就是原生webdriver的子类，另外Remote()构造函数的第一个参数中需要显示指定appium server监听的端口

### java篇

**新建java项目时候，请注意将selenium-webdriver以及appium client的jar包导入**

``` java 
import io.appium.java_client.AppiumDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;

DesiredCapabilities capabilities = new DesiredCapabilities();
capabilities.setCapability(CapabilityType.BROWSER_NAME, "");//这句不是必须的
capabilities.setCapability("deviceName","Android Emulator");
capabilities.setCapability("platformVersion", "4.4");
capabilities.setCapability("platformName","Android");
capabilities.setCapability("appPackage", "com.android.calculator2");
capabilities.setCapability("appActivity", ".Calculator");
AppiumDriver driver = new AppiumDriver(new URL("http://127.0.0.1:4723/wd/hub"), capabilities);
```
 

讨论:AppiumDrvier是原生webdriver的子类。

在这里我们可以看到，新建driver的时候必须要指定一个DesiredCapabilities 对象，该对象究竟是何方神圣，我们下一节会仔细讲解。

### [appium简明教程（7）——Desired Capabilities详解](http://www.cnblogs.com/nbkhic/p/3805805.html)

 

Desired Capabilities在启动session的时候是必须提供的。

Desired Capabilities本质上是key value的对象，它告诉appium server这样一些事情：

*   本次测试是启动浏览器还是启动移动设备？
*   是启动andorid还是启动ios？
*   启动android时，app的package是什么？
*   启动android时，app的activity是什么？
本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途

##### Appium的Desired Capabilities是扩展了webdriver的Desired Capabilities的，下面的一些通用配置是需要指定的：

*   automationName：使用哪种自动化引擎。appium（默认）还是Selendroid？
*   platformName：使用哪种移动平台。`iOS`, `Android`, or`FirefoxOS？`
*   deviceName：启动哪种设备，是真机还是模拟器？`iPhone Simulator`, `iPad Simulator`, `iPhone Retina 4-inch`, `Android Emulator`, `Galaxy S4`, etc...
*   app：应用的绝对路径，注意一定是绝对路径。如果指定了appPackage和appActivity的话，这个属性是可以不设置的。另外这个属性和browserName属性是冲突的。
*   browserName：移动浏览器的名称。比如Safari' for iOS and 'Chrome', 'Chromium', or 'Browser' for Android；与app属性互斥。
*   udid：物理机的id。比如1ae203187fc012g。

##### 下面这些属性是android平台特定的：

*   appActivity：待测试的app的Activity名字。比如MainActivity, .Settings。注意，原生app的话要在activity前加个"."。
*   appPackage：待测试的app的java package。比如com.example.android.myApp, com.android.settings。
本文主要讨论android平台的appium测试方法和技巧，因此在这里就不列出ios设备特定的属性了。

更多信息请参考[官方文档](https://github.com/appium/appium/blob/master/docs/en/caps.md)

在这里我们发现，我们经常要获取app的package和activity名字，那么有什么工具可以让我们方便的获取到这些信息呢？下一节讲回答这个问题。

 

### [appium简明教程（8）——那些工具](http://www.cnblogs.com/nbkhic/p/3806886.html)

 

#### monitor.bat（hierarchyviewer.bat已经不赞成继续使用了）

该文件位于your_andriod_sdk_pathtools下面。以乙醇的机器为例，其位于E:adt-bundle-windows-x86-20131030sdktools下。

该工具可以帮我们找到android控件的content-description，为以后的`find_element_by_accessibility_id` 定位方法做参数使用。

关于什么是content-description，可以参考[官方文档](http://developer.android.com/training/accessibility/accessible-app.html)。

**本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途**

好，露个脸。

![](/image/appium1.png)

#### uiautomatorviewer.bat

该文件位于your_andriod_sdk_pathtools下面。以乙醇的机器为例，其位于E:adt-bundle-windows-x86-20131030sdktools下。

该工具主要用来查看控件的属性，比如resource id，class name等。

该工具也可查看被测app的appPackage（Desired Capabilities中使用）。

爆照。

![](/imageappium2.png)

好了，是不是感觉还缺了点什么呢？

确实如此，被测app的appActivity怎么获取呢？

下一讲我们详细讲解如何获取被测app的appActivity。

### [appium简明教程（9）——如何获取android app的Activity](http://www.cnblogs.com/nbkhic/p/3806951.html)

 

有时候在appium的Desired Capabilities中需要指定被测app的appActivity，下面的方法可能会对你有所帮助。

#### 方法一

如有你有待测项目的源码，那么直接查看源码就好。如果没有，那么请联系有源码的同学，这是推荐方法。

**本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途**

#### 方法二

如果你没有代码，那么可以反编译该app。

这里将用到2个工具，分别是dex2jar和jd-gui。你可以在[这里下载](http://pan.baidu.com/s/1bnlHYyz)目前为止的最新版本以及示例apk。

我们以工具包里的ContactManager.apk为例，简单介绍一下反编译的流程。

*   1，重命名ContactManager.apk为ContactManager.zip并解压得到文件classes.dex；
*   2，解压dex2jar-0.0.9.15.zip，并从命令行进入该文件夹；
*   3，运行命令
<pre>d2j-dex2jar.bat path_toclasses.dex</pre>
在当前文件夹下得到classes-dex2jar.jar；

*   4，解压jd-gui-0.3.6.windows.zip得到文件jd-gui.exe；
*   5，使用jd-gui.exe打开classes-dex2jar.jar；
嗯，好了，可以尽情欣赏了。上图。

![](/image/appium3.png)

上图所示的ContactManager就是待测app的main activity。

#### 方法三

参考testerhome的[这个帖子](http://testerhome.com/topics/1030)

使用log查看大法(嗯，windows上没grep不幸福，好在有powershell的Select-String，可以拿来勉强一用)，直接搬砖。
> a、启动待测apk
> 
> b、开启日志输出：adb logcat>D:/log.txt
> c、关闭日志输出：ctrl+c
> 
> d、查看日志
> 
> 找寻：
> <pre>Displayed com.mm.android.hsy/.ui.LoginActivity: +3s859ms
> appPackage = com.mm.android.hsy
> appActivity = .ui.LoginActivity</pre>
好了，准备活动做的差不多了。下一节乙醇带大家进行控件定位之旅。

### [appium简明教程（10）——控件定位基础](http://www.cnblogs.com/nbkhic/p/3807871.html)

 

狭义上讲，UI级的自动化测试就是让机器代替人去点来点去的过程。

但机器去点什么（点上面还是点左边），怎么点（是长按还是轻触），这些东西是必须由代码的编写者所指示清楚的。

控件定位就是解决机器点什么的问题的。

一般说来，我们可以这样告诉机器：去点**登陆按钮**。

机器很笨，它并不知道什么是**登陆按钮**。因为**登陆按钮**是自然语言的描述。

如果你让一个人去点登陆按钮，那么他其实也是要经过一系列的脑补以后才可以做这件事的。

这个脑补的过程还原如下：
> 这个一定是个按钮
> 
> 这个按钮一定在被测的应用上
> 
> 这个按钮大概上面有**登陆**这个文字信息
> 
> 嗯，还真有一个，那么点吧。
这就是人探索性测试的一个简单过程。一般来说，如果你给出的信息不太充分，人类还是可以通过一系列的探索性思维去理解你的描述的。这个属于心理学的问题，不展开解释。

但是机器并不是人，如果你给出的描述不精确的话，机器是不会自发性的进行探索和脑补的。

因此控件定位就是精确的描述控件特征并告诉机器的过程。

**本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途**

控件的特征就是控件的属性，我们可以通过上一讲中的uiautomatorviewer去获取。

在appium的定位世界里，下面这些方法是可以为我们使用的。也就是说，我们通过下面几个约定好的方式，按照webdriver和appium的DSL（自行搜索并理解）进行控件特征的描述和定位。

继承自webdriver的方法，也就是通过这3个特征可以定位控件

*   find by "class" (i.e., ui component type，andorid上可以是android.widget.TextView)
*   find by "xpath" (i.e., an abstract representation of a path to an element, with certain constraints，由于appium的xpath库不完备的原因，这个不太推荐)
*   find by "id"(android上是控件的resource id)
由[Mobile JSON Wire Protocol](https://code.google.com/p/selenium/source/browse/spec-draft.md?repo=mobile) 协议中定义的方法，更适合移动设备上的控件定位

*   `-ios uiautomation`: a string corresponding to a recursive element search using the UIAutomation library (iOS-only)
*   `-android uiautomator`: a string corresponding to a recursive element search using the UiAutomator Api (Android-only)
*   `accessibility id`: a string corresponding to a recursive element search using the Id/Name that the native Accessibility options utilize.
在appium 的client对[Mobile JSON Wire Protocol](https://code.google.com/p/selenium/source/browse/spec-draft.md?repo=mobile)中定义的方法进行了封装，使其调用起来更加方便

#### ruby篇

``` ruby
find_element :accessibility_id, 'Animation'
find_elements :accessibility_id, 'Animation'
find_element :uiautomator, 'new UiSelector().clickable(true)'
find_elements :uiautomator, 'new UiSelector().clickable(true)'
```

当然了，你也可以使用原生的webdriver方法
> find_element id: 'resource_id'

另外，ruby lib里提供了一些非常好用的简便方法来进行控件的定位，好写，好读。

*   text(value_or_index) :Find the first TextView that contains value or by index. If int then the TextView at that index is returned.
*   button(value_or_index):Find the first button that contains value or by index. If int then the button at that index is returned
更多请看[这里](https://github.com/appium/ruby_lib/blob/master/docs/android_docs.md)

#### python篇

``` python 
el = self.driver.find_element_by_android_uiautomator('new UiSelector().description("Animation")')
self.assertIsNotNone(el)
els = self.driver.find_elements_by_android_uiautomator('new UiSelector().clickable(true)')
self.assertIsInstance(els, list)

el = self.driver.find_element_by_accessibility_id('Animation')
self.assertIsNotNone(el)
els = self.driver.find_elements_by_accessibility_id('Animation')
self.assertIsInstance(els, list)

```


总的来说就是在driver里增加了

*   find_element_by_accessibility_id
*   find_elements_by_accessibility_id
*   find_element_by_android_uiautomator
*   find_element_by_android_uiautomator
等方法

#### java篇

前面也讲过了，新增了这些方法
``` java
findElementByAccessibilityId()
findElementsByAccessibilityId()
findElementByIosUIAutomation()
findElementsByIosUIAutomation()
findElementByAndroidUIAutomator()
findElementsByAndroidUIAutomator()
```

讨论：从上面可以看出来，python 和 java client对移动端控件定位的封装是比较初级的。ruby lib中封装了很多方便和简洁的方法，因此可以看出，使用ruby lib是优于python和java的选择。当然，如果忽略性能的话。

下一节我们开始具体看下如何用resource id去定位控件。

### [appium简明教程（11）——使用resource id定位](http://www.cnblogs.com/nbkhic/p/3813792.html)

 

上一节乙醇带大家了解了appium的定位策略。实际上appium的控件定位方式是完全遵守webdriver的mobile扩展协议的。

这一节将分享一下如何使用resource id来定位android策略。

什么是resource id，这个不属于本文的范畴，大家可以点[这里](http://developer.android.com/guide/topics/resources/accessing-resources.html)了解。

我们可以有两种方式来使用resource id进行定位：

*   使用 findElement(By.id("resourceId")) 的方式。这也是原生的webdriver定义的方法，不过竟然在appium的官方文档里没有提及，属于隐藏技；
*   使用 find_elements_by_android_uiautomator('new UiSelector().resourceId("the_id")') 的方式；关于uiautomator定位后面的教程会展开讲解；
从上面的代码片段可以看到，使用 find_element_by_id 的方式进行定位是最简便的。

那么怎么获取控件的resource id呢，使用uiautomatorviewer就可以了。具体方法如下图所示。

![](/image/appium4.gif)

现在就以上图所示的android原生计算器程序为例，看一下每种语言是如何实现点击【9】这个按钮的。

##### 目的

点击计算器上的【9】这个按钮。该按钮的id是`com.android.calculator2:id/digit6` 。先甜后苦，从ruby开始。

**本文版权归乙醇所有，欢迎转载，但请注明作者与出处，严禁用于任何商业用途**

##### Ruby篇

``` ruby
require 'appium_lib'
caps   = { caps:{ platformName: 'Android', appActivity: '.Calculator', appPackage: 'com.android.calculator2' },
           appium_lib: { sauce_username: nil, sauce_access_key: nil, debug: true} }
dr = Appium::Driver.new(caps).start_driver
dr.find_element(id: 'com.android.calculator2:id/digit9').click
```

##### Python篇

``` python
#coding:utf-8
from appium import webdriver
from time import sleep

desired_caps = {}
desired_caps['platformName'] = 'Android'
desired_caps['platformVersion'] = '4.4'
desired_caps['deviceName'] = 'Android Emulator'
desired_caps['app'] = 'Calculator.apk'
desired_caps['appPackage'] = 'com.android.calculator2'
desired_caps['appActivity'] = '.Calculator'

dr = webdriver.Remote('http://localhost:4723/wd/hub', desired_caps)
sleep(3)

dr.find_element_by_id('com.android.calculator2:id/digit9').click()
```


##### Java篇
``` java
//新建一个FindById类位于info.itest.www package下面
package info.itest.www;

import io.appium.java_client.AppiumDriver;
import java.net.MalformedURLException;
import java.net.URL;
import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;

public class FindById {
    public static void main(String args[]) throws MalformedURLException {
        DesiredCapabilities cap = new DesiredCapabilities();
        cap.setCapability(CapabilityType.BROWSER_NAME, "");
        cap.setCapability("platformName", "Android");
        cap.setCapability("deviceName", "Android Emulator");
        cap.setCapability("platformVersion", "4.4");
        cap.setCapability("appPackage", "com.android.calculator2");
        cap.setCapability("appActivity", ".Calculator");

        AppiumDriver dr = new AppiumDriver(new URL("http://127.0.0.1:4723/wd/hub"), cap);

        dr.findElement(By.id("com.android.calculator2:id/digit9")).click();
    }
}
```

如果读者对webdriver很熟悉的话，那么掌握这个方法是非常简单的。如果对webdriver不熟悉，那么可以参考[乙醇的webdriver实用指南](https://github.com/easonhan007/webdriver_guide/blob/master/README.md)，先学习一下webdriver的基础知识。

这一节我们写了一些脚本去进行控件定位，在实际的项目中，这些没有任何断言的脚本是基本上无法完成测试用例的功能的。

先卖个关子，下下一节乙醇将会带大家写第一个appium的测试用例。

那么下一节我们将学习如何使用class name进行定位。