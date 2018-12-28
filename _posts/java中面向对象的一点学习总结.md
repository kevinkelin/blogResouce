title: java中面向对象的一点学习总结
id: 1108
permalink: 1108
categories:
  - 计算机相关
date: 2014-08-24 23:33:40
tags:
  - java
---

最近开始看java的一些东西，感觉比python麻烦些，今天学习了面向对象的一些东西，觉得挺多挺复杂，这里做个知识总结

以一个简单的例子来说明java面向对象的三大特性，封装，继承，多态，有一个动物(Animal)基类，定义了run与eat方法，然后有一个猫(Cat)与狗(Dog)的子类继承了动物这个父类，子类重写(override)了父类的run与eat方法，同步又重载(overload)了run与eat方法，同时又定义了一个交配(Icopulation)的接口，让狗来implements交配接口，同时又创建一个Human类来专门implements交配接口

[![795ab47fjw1ejo4u6epeoj20ab083jrh](/image/2014/08/795ab47fjw1ejo4u6epeoj20ab083jrh_thumb.jpg "795ab47fjw1ejo4u6epeoj20ab083jrh")](/image/2014/08/795ab47fjw1ejo4u6epeoj20ab083jrh.jpg) 

下面是具体的实现代码，没有什么实际的应用，只是作为学习用
<!-- more -->
Animal.java 这个是父类
``` java
package com.yangyanxing.www;
//这里是定义了一个Animal的基类
public class Animal {
public void run(){
System.out.println("我是所有动物的跑");
}
public void eat(){
System.out.println("我是所有动物在吃");
}
}
```

Icopulation.java 交配的接口

``` java
package com.yangyanxing.www;
//这里定义了一个接口类，动物应该都具有交配
public interface Icopulation {
public void copulation();//这里是一个规范，要有交配方法，具体怎么实现要子类去实现
}
```

&#160;

Dog.java Dog的子类

``` java
package com.yangyanxing.www;
//这里定义了一个狗的类继承Animal基类
public class Dog extends Animal implements Icopulation {
//对父类的方法进行重写(override) 方法的返回值类型与参数都不能变
public void run(){
System.out.println("我是一条狗在跑");
}
//方法的重载(overload)只是参数的数量与类型不同，返回值与权限都要相同
//这里由于父类里的run方法没有重载，所以使用多态创建的子类引用也不能使用带参数的run(5)
public void run(int a){
System.out.printf("我是只狗，已经跑了%d公里了n",a);
}
public void eat(){
System.out.println("我是一条狗是吃");
}
//这里是狗自已的方法，不是从基类继承的
public void creame(){
System.out.println("这是一条狗在叫");
}
public void copulation(){
System.out.println("狗在交配");
}
}
```

&#160;

Cat.java Cat子类

``` java
package com.yangyanxing.www;

public class Cat extends Animal {
public void run(){
System.out.println("我是一只猫在跑");
}
public void eat(){
System.out.println("我是一条猫是吃");
}
public void eat(String food){
System.out.printf("我是只猫，我正在吃%sn",food);
}
//这里是猫自已的方法，不是从基类继承的
public void creame(){
System.out.println("这是一条猫在叫");
}
}

```

&#160;

Human.java 实现了交配的Human类

``` java
package com.yangyanxing.www;

public class Human implements Icopulation {

@Override
public void copulation() {
// TODO Auto-generated method stub
System.out.println("人在交配");

}

}

```

Testoob.java 具体的测试代码

``` java
package com.yangyanxing.www;
//这里不用import 引用，在同一个包里
public class Testoob {

public static void main(String[] args) {
// TODO Auto-generated method stub
Animal a  = new Animal();
Animal a1 = new Dog();//多态，使用父类创建子类的引用
Animal a2 = new Cat();
Dog dog = new Dog();
Cat cat = new Cat();
a.run();
a.eat();
a1.run();
a1.eat();
//a1.creame();//多态父类创建的子类不能使用子类自已的方法
a2.run();
a2.eat();
dog.creame();//不是通过多态创建的子类可以使用自已的方法
cat.creame();
cat.eat("鱼");
dog.run(5);
//a2.eat("fish");//这里会出错，因为父类里没有定义带参数的eat()方法

Icopulation idog = new Dog();
idog.copulation();

Icopulation ihuman = new Human();
ihuman.copulation();
}

}

```

以下是程序运行的输出结果

我是所有动物的跑

我是所有动物在吃

我是一条狗在跑

我是一条狗是吃

我是一只猫在跑

我是一条猫是吃

这是一条狗在叫

这是一条猫在叫

我是只猫，我正在吃鱼

我是只狗，已经跑了5公里了

狗在交配

人在交配