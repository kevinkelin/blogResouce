title: spinner和适配器模式
id: 1310
permalink: 1310
categories:
  - Android
date: 2015-02-22 00:41:48
tags:
  - android
---

原文 [http://www.cnblogs.com/UUUP/p/3983394.html](http://www.cnblogs.com/UUUP/p/3983394.html "http://www.cnblogs.com/UUUP/p/3983394.html")

spinner相当于html表单中的select下拉列表。

  
# 第一种方式    
  
在string.xml中添加一个数组spinner_data：
``` xml    
<?xml version="1.0" encoding="utf-8"?> 
<resources> 
<string-array name="spinner_data"> 
<item >北京</item> 
<item >湖南</item> 
<item >湖北</item> 
</string-array> 
</resources>
```
<!-- more -->
拖拽一个spinner：
``` xml   
<Spinner 
android:id="@+id/spinner1" 
android:layout_width="match_parent" 
android:layout_height="wrap_content" 
android:entries="@array/spinner_data" 
android:spinnerMode="dialog" 
/>
<!-- entries 显示spinner当中的数据项 
spinnerMode="dropdown"是默认值下拉列表，spinnerMode="dialog"是以对话框的形式显示--!> 

```

 
# 第二种方式   
  
BaseAdapter就Android应用程序中经常用到的基础数据适配器，它的主要用途是将一组数据传到像ListView、Spinner、Gallery、GridView等UI显示组件，它是自动继承接口类Adapter。    
假如要往spinner中添加string[]或List。       
a）string[]    
在MainActivity主类中定义一个字符串数组：    
private String[] str = new String[] {    
 "山东","山西","北京"    
};

定义一个继承BaseAdapter的类：   
private class MyAdapter extends BaseAdapter {

@Override   
public int getCount() {    
 // TODO Auto-generated method stub    
 return str.length;//重要方法    
}

@Override   
public Object getItem(int arg0) {    
 // TODO Auto-generated method stub    
 return null;    
}

@Override   
public long getItemId(int arg0) {    
 // TODO Auto-generated method stub    
 return 0;    
}

@Override   
public View getView(int position, View view, ViewGroup group) {//重要方法    
 // TODO Auto-generated method stub    
 //str.length多长此方法就执行几次    
 TextView textView = new TextView(MainActivity.this);    
 textView.setText(str[position]);    
 return textView;    
}    
}

在onCreate方法中：   
Spinner spinner = (Spinner) findViewById(R.id.spinner1);     
spinner.setAdapter(new MyAdapter());       
b)List    
在MainActivity主类中定义一个List：    
private List list = new ArrayList();

并在onCreate方法中添加内容：   
list.add("上海");    
list.add("天津");    
list.add("浙江");

MyAdapter类则只需要修改几处即可：   
1\. public int getCount() {

 return list.size();   
}

2\. public View getView(int position, View view, ViewGroup group) {

 TextView textView = new TextView(MainActivity.this);   
 textView.setText((CharSequence) list.get(position));    
 return textView;    
}

3.还可以继续给spinner添加事件OnItemSelectedListener：   
spinner.setOnItemSelectedListener(new OnItemSelectedListener() {

@Override   
public void onItemSelected(AdapterView<?> parent, View view,    
int position, long id) {    
// TODO Auto-generated method stub    
 Toast.makeText(MainActivity.this, (CharSequence) list.get(position), 0).show();    
}

@Override   
public void onNothingSelected(AdapterView<?> parent) {    
// TODO Auto-generated method stub    
}    
});

&#160;

&#160;

# 利用ArrayAdapter构造adapter
``` java

//第一步：添加一个下拉列表项的数据源ss，这里添加的项就是下拉列表的菜单项
private String[] ss = new String[] { "云南", "北京", "香港" };
Spinner spinner=(Spinner) findViewById(R.id._spinner1_);

//第二步：为下拉列表定义一个适配器，这里就用到里前面定义的ss。

ArrayAdapter<String> adapter= new ArrayAdapter<String>(this,android.R.layout._simple_spinner_item_, ss);

//其中，第一个是conetxt，也就是application的环境，第二个参数是spinner未展开的布局方式，第三个参数是数据源

//第三步：为适配器设置下拉列表下拉时的菜单样式。

adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);

//第四步：将适配器添加到下拉列表上

spinner.setAdapter(adapter);

//第五步：为下拉列表设置各种事件的响应，这个事响应菜单被选中

spinner.setOnItemSelectedListener(new OnItemSelectedListener){

publicvoid onItemSelected(AdapterView<?> parent,View view, int position, long id) {

Toast._makeText_(MainActivity.this,ss[position], 0).show();

}
```