title: Uiautomator中watcher的使用
categories: 
  - Android
date: 2016-05-13 23:09:30
tags: 
  - Android
  - 自动化测试
permalink: use-watcher-in-uiautomator

---

今天在uiautomator中实践了watcher的用法，这个也是之前在python中使用uiautomator中比较喜欢的功能，它可以提前定义一些条件，当满足一些条件时，进行一些操作，这个常用于处理测试过程中某些意料之外的或者不知道什么时候弹出来的框而阻碍测试的正常进行。
之前在写自动化用例的时候，遇到过小米手机在安装app的时候，会弹一个框来让用户点击安装，还有弹出一个升级检测的框点击“取消”按钮，或者遇到退出的时候点击确定，当然这些完全可以在用例里写逻辑来处理，而且有些还是程序本身要测试的检测点，当然这些对于大多数测试来说没有太大的意义，所以可以将其放入一个watcher里来让uiautomator来帮你进行相应的点击处理。
<!-- more -->
查看官方文档， UiWatcher是由UiDevice来registerWatcher (String name, UiWatcher watcher)，name为一个名字，这个名字相当于一个key,可以在之后的查看该watcher是否被检测到了使用和remove的时候使用

初始化UiWatcher时要重写一个checkForCondition()这个抽象方法，这个方法主要就是写一些判断哪些UiSelector是否出现了，出现了怎么处理，我重新封装了一个方法

``` java
public void initwatch(final String name,final UiSelector checkSelecto,final UiSelector opSelector){
        //将name添加到watcherNames的list中，为了tearDown方法中remove掉用
        watcherNames.add(name);
        final UiObject check = new UiObject(checkSelecto);
        final UiObject op = new UiObject(opSelector);
        mDevice.registerWatcher(name, new UiWatcher() {
            //这里重写checkForCondition方法
            public boolean checkForCondition() {
                try {
                    if (check.exists()) {
                        op.click();
                        Thread.sleep(1000);
                        return true;
                    }
                    else{
                        Thread.sleep(1000);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
                
                return false;
            }
        });
    }

```

checkForConditon方法的返回值是boolean值，当返回true的时候，则uidevice.hasWatcherTriggered(name) 则返回true，checkForConditon这里不用写循环，uiautomator会在测试过程中一直在循环的调用。

以下是全部代码
TestUtil.java

``` java
package com.yangyanxing.test;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.util.ArrayList;

import com.android.uiautomator.core.UiDevice;
import com.android.uiautomator.core.UiObject;
import com.android.uiautomator.core.UiSelector;
import com.android.uiautomator.core.UiWatcher;

public class TestUtil {
    public ArrayList<String> watcherNames = new ArrayList<String>(); 
    public UiDevice mDevice = UiDevice.getInstance();
    
    public static String doCmdshell(String commond){
        String s = null;
        try
        {
            Process p = Runtime.getRuntime().exec(commond);
            BufferedReader stdInput = new BufferedReader(new InputStreamReader(p.getInputStream()));
            BufferedReader stdError = new BufferedReader(new InputStreamReader(p.getErrorStream()));

            String result = "";
            while ((s = stdInput.readLine()) != null)
            {
                result = result + s + "\n";
            }
            while ((s = stdError.readLine()) != null)
            {
                System.out.println(s);
            }
            return result;
        }
        catch (Exception e)
        {
            return "Exception occurred";
        }
    }
    //init watcher 
    
    
    public void initwatch(final String name,final UiSelector checkSelecto,final UiSelector opSelector){
        //将name添加到watcherNames的list中，为了tearDown方法中remove掉用
        watcherNames.add(name);
        final UiObject check = new UiObject(checkSelecto);
        final UiObject op = new UiObject(opSelector);
        mDevice.registerWatcher(name, new UiWatcher() {
            //这里重写checkForCondition方法
            public boolean checkForCondition() {
                try {
                    if (check.exists()) {
                        op.click();
                        Thread.sleep(1000);
                        return true;
                    }
                    else{
                        Thread.sleep(1000);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
                
                return false;
            }
        });
    }
    
    
    public static Boolean waitForUiselectorAppears(UiSelector selector,int timeout)
    {
        UiObject uiObject = new UiObject(selector);
        return uiObject.waitForExists(timeout*1000); 
    }

}
```

UitestRunner.java

``` java
package com.yangyanxing.test;

import com.android.uiautomator.core.UiDevice;
import com.android.uiautomator.core.UiObject;
import com.android.uiautomator.core.UiObjectNotFoundException;
import com.android.uiautomator.core.UiSelector;
import com.android.uiautomator.testrunner.UiAutomatorTestCase;
import static com.yangyanxing.test.TestUtil.doCmdshell;


public class UitestRunner extends UiAutomatorTestCase {
    //初始化一个UiDevice
    private UiDevice mDevice = UiDevice.getInstance();
    private TestUtil tUtil = new TestUtil();
    
    public UitestRunner(){
        super();
    }   
    //写setUp()方法
    public void setUp() throws Exception{
        super.setUp();
        //每次测试的时候都需要启动急救箱，所以将这个方法放到setUp里
        doCmdshell("am start com.qihoo.mkiller/com.qihoo.mkiller.ui.index.AppEnterActivity");       
        System.out.println("开始测试啦。。。");
        //注册一些watcher
        tUtil.initwatch("ignorSafe", new UiSelector().textContains("安全上网").className("android.widget.TextView"), 
                new UiSelector().text("取消"));
        tUtil.initwatch("agree", new UiSelector().text("同意并使用"), 
                new UiSelector().text("同意并使用"));
        tUtil.initwatch("Noupdate", new UiSelector().textContains("升级"), 
                new UiSelector().text("取消"));
        mDevice.runWatchers();//将watchers运行起来
    }
    
    //写tearDown方法，将急救箱force-stop
    public void tearDown() throws Exception{
        super.tearDown();
        doCmdshell("am force-stop com.qihoo.mkiller");
        System.out.println("用例测试完了！");
        for (String watcherName : tUtil.watcherNames) {
            System.out.println(watcherName+"被remove了！");
            mDevice.removeWatcher(watcherName);
        }
    }
    
    //检测急救箱启动后是否有"开始扫描"按钮

    public void test_startScanButton() throws UiObjectNotFoundException{
        UiSelector scanButton = new UiSelector().className("android.widget.Button").text("开始扫描");
        if(TestUtil.waitForUiselectorAppears(scanButton, 20)){
            UiObject scanoObject = new UiObject(scanButton);
            if (scanoObject.click()) {
                System.out.println("开始扫描 按钮被点击了！");
            }else{
                System.out.println("开始扫描 按钮点击失败了");
            }
        }else {
            System.out.println("急救箱启动失败");
        }
        
        UiSelector exitbutton = new UiSelector().className("android.widget.Button").text("退出");
        
        assertEquals(Boolean.TRUE, TestUtil.waitForUiselectorAppears(exitbutton, 120));
    }
    
    public void test_print(){
        System.out.println("用例2开始测试了！");
    }
    
}
```



