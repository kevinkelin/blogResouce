title: Android中的listView与ArrayAdapter与LayoutInflater的使用
id: 1385
permalink: 1385
comment: false
categories:
  - Android
date: 2015-10-06 01:21:03
tags:
  - android
---

最近在看《第一行代码-android》，这本书讲的不错，从最android基础的开始讲起，由浅入深，一步一步的教怎么使用android开发中的各种内容，今天看到listView，书中讲到listView可能是使用最多也是最难的一个组件，看过之后觉得还是需要好好消化一下的，借助书中的代码，来记录一下学习的过程

一、在listView中简单的显示一行文字

这个应该是listView应用中最简单的了，在使用listView中，一般的步骤应该是，先在main_activity.xml中创建好listView的布局
```  html
LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:orientation="vertical" >

<ListView
    android:id="@+id/list_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

</ListView>

</LinearLayout>
```
<!-- more -->
之后在MainActivity.java中构造数据，这里应该是从数据库或者网上来获得数据，而为了方便只是简单的使用了一个数组

`private String[] data = { "Apple", "Banana", "Orange", "Watermelon", "Pear", "Grape", "Pineapple", "Strawberry", "Cherry", "Mango" };`


之后再使用一个arrayAdapter来适配array与listView

``` java
ArrayAdapter<String> adapter = new    ArrayAdapter<String>(MainActivity.this,    android.R.layout.simple_list_item_1,
```

关于arrayAdapter的使用，[http://www.cnblogs.com/loulijun/archive/2011/12/26/2302287.html](http://www.cnblogs.com/loulijun/archive/2011/12/26/2302287.html "http://www.cnblogs.com/loulijun/archive/2011/12/26/2302287.html") 这篇文章介绍的还算比较简单

ArrayAdapter的初始化有多种方法，上面的应该是最简单通用的方法，第一个参数是一个上下文，一般是this,第二个布局文件，这里使用系统自带的android.R.layout.simple_list_item_1，后面有介绍如何使用自定义的布局文件，第三个参数是数据源。

将适配器适配到listView中

MainActivity.java的完整代码如下

```  java
package com.example.uilayouttest;

import java.util.ArrayList;
import java.util.List;

import android.app.Activity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.Window;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;

public class MainActivity extends Activity {

private String[] data = { "Apple", "Banana", "Orange", "Watermelon", "Pear", "Grape", "Pineapple", "Strawberry",
"Cherry", "Mango" };

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
requestWindowFeature(Window.FEATURE_NO_TITLE);
setContentView(R.layout.activity_main);

ArrayAdapter<String> adapter = newArrayAdapter<String>(MainActivity.this,android.R.layout.simple_list_item_1,
            data);
ListView listView = (ListView) findViewById(R.id.list_view);
listView.setAdapter(adapter);

}

@Override
public boolean onCreateOptionsMenu(Menu menu) {
// Inflate the menu; this adds items to the action bar if it is present.
getMenuInflater().inflate(R.menu.main, menu);
return true;
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
// Handle action bar item clicks here. The action bar will
// automatically handle clicks on the Home/Up button, so long
// as you specify a parent activity in AndroidManifest.xml.
int id = item.getItemId();
if (id == R.id.action_settings) {
return true;
}
return super.onOptionsItemSelected(item);
}
}
```


二、使用自定义的ListView

上面这个应该说是最简单的，下面来说说自定义的

如果说如果想要在listView和每个项(item)里包含一张图片与一个文字，那么就需要自定义listView了

首先先在layout下自定义一个xml文件,fruit_item.xml

```  html
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <ImageView
        android:id="@+id/fruit_image"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/fruit_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:layout_marginLeft="10dip" />

```


这个布局文件其实就是定义了每个项的布局，之后创建一个数据源，它这里是使用了一个ArrayLlist，并创建了一个Fruit类，在初始化ArrayList的时候采用泛型限制只能用Fruit类。

Fruit类定义如下

```  java
package com.example.uilayouttest;

public class Fruit {
private String name;

public Fruit(String name){
this.name = name;

}

public String getName(){
return this.name;
}

public int getImageId() {
return R.drawable.ic_launcher;
}

}
```

由于我没有那么多的图片，所以我就只返回了一张默认的ic_launcher的图片

自定义一个FruitAdapter类继承自ArrayAdapter类

```  java
package com.example.uilayouttest;

import java.util.List;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.ImageView;
import android.widget.TextView;

public class FruitAdapter extends ArrayAdapter<Fruit> {
private int resourceId;

public FruitAdapter(Context context, int textViewResourceId, List<Fruit> objects) {
super(context, textViewResourceId, objects);
resourceId = textViewResourceId;

}

// 这个方法在每个子项被滚动到屏幕内的时候 会被调用
public View getView(int position, View convertView, ViewGroup parent) {
Fruit fruit = getItem(position);// getItem()方法得到当前项的 Fruit 实例
View view;
ViewHolder viewHolder;
if (convertView == null) {
view = LayoutInflater.from(getContext()).inflate(resourceId, null);
viewHolder = new ViewHolder();
viewHolder.fruitImage = (ImageView) view.findViewById(R.id.fruit_image);
viewHolder.fruitName = (TextView) view.findViewById(R.id.fruit_name);
view.setTag(viewHolder);
} else {
view = convertView;
viewHolder = (ViewHolder) view.getTag();
}
viewHolder.fruitImage.setImageResource(fruit.getImageId());
viewHolder.fruitName.setText(fruit.getName());
return view;

}

class ViewHolder {
ImageView fruitImage;
TextView fruitName;
}

}
```

这里有一些要记录的，首先初始化

public FruitAdapter(Context context, int textViewResourceId, List<Fruit> objects) {

    super(context, textViewResourceId, objects);

    resourceId = textViewResourceId; 

}



这里有三个参数，其实和ArrayAdapter的简单初始化一样，但是这里的第二个参数，在之后的初始化时，要传入的就是刚才自定义的fruit_item.xml布局文件。

第三个参数是数据源。

在FruitAdapter适配器的定义中使用到了LayoutInflater

view = LayoutInflater.from(getContext()).inflate(resourceId, null);

网上查了查，这个是加载布局文件的函数，之前一直都是使用setContentView(R.layout.activity_main);

其实这个是更深入的函数

[http://blog.csdn.net/guolin_blog/article/details/12921889](http://blog.csdn.net/guolin_blog/article/details/12921889 "http://blog.csdn.net/guolin_blog/article/details/12921889"); 这彷文章介绍的比较详细

比如说要在A布局上加载B布局，那么先初始化LayoutInflater的实例

LayoutInflater.from(getContext())，from里的参数就是A，这里就直接使用getContent()，然后再使用inflate(resourceId, null)加载B布局

这里由于在FruitAdapter初始化的时候将resourceId = textViewResourceId;也就是说这里是将自定义的fruit_item布局加载到主的main_activity中

之后为了性能的优化，使用了ViewHolder与convertView，convertView这个参数用于将之前加载好的布局进行缓存，以便之后可以进行重用

最终的MainActivity.java如下

```  java
package com.example.uilayouttest;

import java.util.ArrayList;
import java.util.List;

import android.app.Activity;
import android.os.Bundle;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.Window;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;

public class MainActivity extends Activity {

private List<Fruit> frutlist = new ArrayList<Fruit>();

private void initFruit() {
Fruit apple = new Fruit("apple");
Fruit banana = new Fruit("banane");
Fruit orange = new Fruit("orange");
Fruit Watermelon = new Fruit("Watermelon");
Fruit Pear = new Fruit("Pear");
Fruit Grape = new Fruit("Grape");
Fruit Strawberry = new Fruit("Strawberry");
Fruit Pineapple = new Fruit("Pineapple");
Fruit Cherry = new Fruit("Cherry");
Fruit Mango = new Fruit("Mango");
frutlist.add(apple);
frutlist.add(banana);
frutlist.add(Mango);
frutlist.add(Cherry);
frutlist.add(Pineapple);
frutlist.add(Strawberry);
frutlist.add(Grape);
frutlist.add(Pear);
frutlist.add(Watermelon);
frutlist.add(orange);
}

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
requestWindowFeature(Window.FEATURE_NO_TITLE);
setContentView(R.layout.activity_main);

initFruit();

FruitAdapter adapter = new FruitAdapter(MainActivity.this, R.layout.fruit_item, frutlist);
ListView listView = (ListView) findViewById(R.id.list_view);
listView.setAdapter(adapter);
listView.setOnItemClickListener(new OnItemClickListener() {

@Override
public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
String name = frutlist.get(position).getName();
Toast.makeText(MainActivity.this, name, Toast.LENGTH_SHORT).show();

}

});

}

@Override
public boolean onCreateOptionsMenu(Menu menu) {
// Inflate the menu; this adds items to the action bar if it is present.
getMenuInflater().inflate(R.menu.main, menu);
return true;
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
// Handle action bar item clicks here. The action bar will
// automatically handle clicks on the Home/Up button, so long
// as you specify a parent activity in AndroidManifest.xml.
int id = item.getItemId();
if (id == R.id.action_settings) {
return true;
}
return super.onOptionsItemSelected(item);
}
}
```