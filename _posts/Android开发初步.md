title: Android开发初步
id: 1303
permalink: 1303
categories:
  - Android
date: 2015-02-19 19:36:23
tags:
  - android
---

绑定单击事件有两种方法，一种是通过绑定android:onClick属性，一种是绑定一个setOnClickListener回调函数

# 通过绑定onClick属性

在activity_main.xml对应的组件上设置android:onClick="test" 属性，然后再Java文件里定义一个test方法来实现，这个test方法是MainActivity类的一个方法
``` javascript

public void test(View view){
        Toast.makeText(MainActivity.this, "你还真敢点啊！", Toast.LENGTH_LONG).show();
    }
    
```

这里的view就是要使用的控件，如果是绑定的是个button，那么view就是这个button
<!-- more -->
# 通过setOnClickListener

此时需要将这个setOnClickListener回调函数写在onCreate()函数里

``` javascript

protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Button button = (Button)findViewById(R.id.button1);
        button.setOnClickListener(new OnClickListener() {           
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "我是被点击的", Toast.LENGTH_LONG).show();               
            }
        });
    }
    
```
<!-- more -->

可以看到setOnClickListener不属于某个控件，在使用的时候要先通过findViewById方法找到要进行onClick方法的控件。

# 将2中的onClickListener方法分离出来

这个方式将OnClickListener分离出来写在MainActibity类方法

``` javascript
private View.OnClickListener myListener = new OnClickListener() {
        @Override
        public void onClick(View v) {
            switch (v.getId()) {
            case R.id.button1:
                Toast.makeText(MainActivity.this, "button1被点击了", Toast.LENGTH_LONG).show();   
                break;
            case R.id.button2:
                Toast.makeText(MainActivity.this, "button2被点击了", Toast.LENGTH_LONG).show();   
                break;
            case R.id.button3:
                Toast.makeText(MainActivity.this, "button3被点击了", Toast.LENGTH_LONG).show();   
                break;
            default:
                break;
            }
        }
    };
    
```

然后在onCreate方法中使用

Button button1 = (Button)findViewById(R.id.button1);   
Button button2 = (Button)findViewById(R.id.button2);    
Button button3 = (Button)findViewById(R.id.button3);    
button1.setOnClickListener(myListener);    
button2.setOnClickListener(myListener);    
button3.setOnClickListener(myListener);

这种方法可以复用方法，也可以多些判断，比较灵活

# 长按与短按

长按有一个返回值，当一个控件既绑定了一个长铵同时又绑定了一个短按，如果长按的返回值是false时，那么在长按以后同时也会触发短按事件
``` javascript

button4.setOnLongClickListener(new OnLongClickListener() {           
            public boolean onLongClick(View v) {
                System.out.println("按钮进行了长按");
                return true; //返回true后就不会再触发短按事件
            }
        });
        button4.setOnClickListener(new OnClickListener() {
            public void onClick(View v) {
                System.out.println("按钮进行了短按");               
            }
        });
        
```
 

# setOnTouchListener事件，滑动事件

``` javascript
final Button button4 = (Button)findViewById(R.id.button4);
        ViewGroup viewGroup = (ViewGroup) findViewById(R.id.vg1);
        viewGroup.setOnTouchListener(new OnTouchListener() {       
            public boolean onTouch(View v, MotionEvent event) {
                int eventType = event.getAction();
                if (eventType == MotionEvent.ACTION_DOWN) {
                    button4.setX(event.getX());
                    button4.setY(event.getY());
                    System.out.println("down…");
                }else if (eventType == MotionEvent.ACTION_MOVE) {
                    button4.setX(event.getX());
                    button4.setY(event.getY());
                    System.out.println("move…");
                }else if (eventType==MotionEvent.ACTION_UP) {
             &
#160;      System.out.println("up…");
                }
                return true;
            }
        });

```

# setOnKeyListener 监听键盘按键操作
``` javascript
button4.setOnKeyListener(new OnKeyListener() {
            @Override
            public boolean onKey(View v, int keyCode, KeyEvent event) {
                //a:29 s:47 d:32 w:51
                if (keyCode == 29) {
                    button4.setX(button4.getX()-10);
                }else if (keyCode == 47) {
                    button4.setY(button4.getY()+10);
                }else if (keyCode == 32) {
                    button4.setX(button4.getX()+10);
                }else if (keyCode == 51) {
                    button4.setY(button4.getY()-10);
                }else {
                }
                return false;
            }
        });
```