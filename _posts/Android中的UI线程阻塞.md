title: Android中的UI线程阻塞
id: 1308
permalink: 1308
categories:
  - Android
date: 2015-02-20 01:05:19
tags:
  - android
---

当一个应用程序启动之后，android系统会为这个应用程序创建一个主线程，这个线程非常重要，它负责渲染视图，分发事件到响应监听器并执行，对界面进行轮询的监听，一般叫做“UI主线程”

Android系统不会给应用程序的多个元素组件建立多个线程来执行，一个视图(activity)中的多个view组件运行在同一个UI线程当中，因此，多个view组件的监听器的执行可能会相互影响。

如有以下两个button，其中一个在会在主view中进行移动动画，另外一个button在点击以后将线程sleep 5秒
<!-- more -->
``` android
Button moveButton = (Button) findViewById(R.id.button5); 
//添加一个animation 
TranslateAnimation animation = new TranslateAnimation(0, 150, 0, 0); 
animation.setRepeatCount(30); 
animation.setDuration(2000); 
moveButton.setAnimation(animation);//将moveButton绑定到这个animation中
 
button4.setOnClickListener(new OnClickListener() { 
            public void onClick(View v) { 
                try { 
                    Thread.sleep(5000); 
                } catch (InterruptedException e) {                    
                    e.printStackTrace(); 
                } 
            }
});

``` 


此时由于button4与moveButton处在同一个UI线程中，button4的Thread.sleep会影响到moveButton的animation，如果UI线程阻塞时间过长，Android系统可能就会干预

[![image](/image/2015/02/image_thumb12.png "image")](/image/2015/02/image12.png) 

而用户也基本上会选择OK来关闭程序。

&#160;

解决线程阻塞问题可以通过将耗时的操作放入一个新的线程中进行，但是会有一个新的问题，在新的线程中又不能对UI线程进行UI操作，于是android官方提供一个post方法来解决这个问题

``` java
button4.setOnClickListener(new OnClickListener() { 
            public void onClick( final View v) { 
                //方法一，新建一个线程，把耗时操作放在这个新线程中进行，而不影响UI线程 
                new Thread(new Runnable() { 
                    public void run() { 
                        try { 
                            Thread.sleep(500); 
                        } catch (InterruptedException e) {                    
                            e.printStackTrace(); 
                        } 
                        //当新线程中试图改变主UI进程的表现时，可以使用view.post()方法 
                        v.post(new Runnable() { 
                            public void run() { 
                                TextView textView = (TextView) v; 
                                textView.setText("yangyanxing"); 
                            } 
                        }); 
                    } 
                }).start(); 
            } 
        });
```