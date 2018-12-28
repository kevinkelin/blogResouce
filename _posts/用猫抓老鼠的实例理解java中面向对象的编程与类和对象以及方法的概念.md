title: 用猫抓老鼠的实例理解java中面向对象的编程与类和对象以及方法的概念
id: 121
permalink: 121
categories:
  - 计算机相关
date: 2011-06-03 02:16:17
tags:
---

今天看到马士兵讲的关于面向对象编程的思路，用了一个猫抓老鼠的例子，我觉得这个例子非常形象，于是写在这里，方便学习理解和以后查看

<!-- more -->

class cat{   //声明一个类--“猫”

int furcolor; // 定义猫的一个属性（成员变量）“毛的颜色”

float heght; // 定义猫的一个属性“身高”

float weight;// 定义猫的一个属性“体重”<span class="Apple-tab-span" style="white-space:pre"> </span>

static void catchmouse(mouse m){ //声明一个方法----“抓老鼠”

//how to catch a mouse //方法里面的关于怎么抓老鼠

//m.scream();

}

 

public static void main(String[] args){// 主函数

cat C = new cat(); //构造一个猫这个类里的一个新“对象”----“C”

mouse M = new mouse();// 构造老鼠里面的一个新“对象”-----“M”

C.catchmouse(M);// 猫的一个方法“抓老鼠”（具体到抓M那只老鼠）<span class="Apple-tab-span" style="white-space:pre"> </span>

}

}

class mouse{ //定义老鼠这个类

void scream(){ //定义老鼠这个类的一个方法scream

System.out.println("mouse");

}<span class="Apple-tab-span" style="white-space:pre"> </span>

}

此程序形象的说明了面向对象中的各种概念，类，对象，方法等