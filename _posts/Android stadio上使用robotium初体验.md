title: Android stadio上使用robotium初体验
id: 1320
permalink: 1320
categories:
  - Android
date: 2015-03-05 22:57:37
tags:
  - android
---

在Android Stadio(as)上创建一个app的时候默认会自动创建相应的test类，可以直接在里面写单元测试用例

[![image](/image/2015/03/image_thumb.png "image")](/image/2015/03/image.png) 

一、在项目(module)中导入robotium的jar包，右键app->new->directory,输入libs

然后将robotium-solo-5.3.1.jar复制进去，然后右键robotium-solo-5.3.1.jar选择add as library,之后就可以写测试用例了

如果还有问题，看一下项目的build gradle
```
dependencies {   
 compile fileTree(include: ['*.jar'], dir: 'libs')    
 compile 'com.android.support:appcompat-v7:21.0.3'    
 compile files('libs/robotium-solo-5.3.1.jar')    
}
```
<!-- more -->

加入 compile files('libs/robotium-solo-5.3.1.jar')   

以下是我简单的写了一个小的测试app和其单元测试,非常简单，没什么实际意义，只是为了演示，其目录结构如下

[![image](/image/2015/03/image_thumb1.png "image")](/image/2015/03/image1.png) 

app代码
  ```  java
package com.example.kevin.helloword;

import android.support.v7.app.ActionBarActivity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

public class MainActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button) findViewById(R.id.button);
        final TextView tx = (TextView)findViewById(R.id.textView);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this,"helloWorld",Toast.LENGTH_SHORT).show();
                Toast.makeText(MainActivity.this,"杨彦星",Toast.LENGTH_LONG).show();
                tx.setText("hello yyx");
            }
        });
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }
}
```

&#160;

其单元的测试代码如下

  ```  java
package com.example.kevin.helloword;

import android.test.ActivityInstrumentationTestCase2;

import com.robotium.solo.Solo;

public class helloTest extends ActivityInstrumentationTestCase2<MainActivity> {

    private Solo solo;

    public helloTest() {
        super(MainActivity.class);

    }

    @Override
    public void setUp() throws Exception {
        //setUp() is run before a test case is started.
        //This is where the solo object is created.
        solo = new Solo(getInstrumentation(), getActivity());
    }

    @Override
    public void tearDown() throws Exception {
        //tearDown() is run after a test case has finished.
        //finishOpenedActivities() will finish all the activities that have been opened during the test execution.
        solo.finishOpenedActivities();
    }

    public void testclickMe() throws Exception{
        solo.unlockScreen();
        solo.clickOnButton("click me");
        boolean expected = true;
        boolean actual = solo.searchText("hello yyx");
        assertEquals("word not change", expected, actual);

    }

}
```
