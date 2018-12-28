title: 'c++语言基础(hello world,数据类型,构造方法,指针的常见错误,常量指针,指针与引用)'
id: 1212
permalink: 1212
categories:
  - C++
date: 2014-12-06 22:13:08
tags:
---

```  cpp
一、hello world
#include<iostream>  这里可以用#include<iostream.h> 加载一个非标准的库，由于.h还没有名字空间的概念，所以之后也就没有使用std的操作
using namespace std; 声明定义使用std 名字空间
namespace a
{
     int b = 5;
}
namespace c{
     int b = 8;
}
int main(){
     std::cout<<"我喜欢C++n";
     std::cout<<"五年一班数学成绩n";
     std::cout<<"第一名许凡的成绩t"<<100;
     std::cout<<std::endl;
     std::cout<<"第二名许凡的成绩t"<<90+8;
     std::cout<<std::endl;
     std::cout<<"第三名许凡的成绩t"<<(float)5/8;
     std::cout<<std::endl;

     cout<<"我是没有使用std的"<<endl;
     int b = 10;

     cout<<b<<" "<<a::b<<" "<<c::b<<endl;

     return 0;
}
```

<!-- more -->

二、数据类型
     1\. 常量 使用const 关键字定义 const name = "yangyanxing";
     2\. 枚举型常量 enum num{zero,one,two,three,four};
``` cpp
     int main()
{
     enum day{Sunday,Monday,Tuesday,Wednesday,Thurday,Friday,Saturday};
     day today; // 使用枚举类型day 创建一个today对象
     today = Monday;// 相当于today=1
     if (today == Sunday||today == Saturday){
          cout<<"今天休息";
     }
     else{
          cout<<"今天工作"<<endl;
     }
     return 0;
}
```


三、构造方法
``` cpp
#include <iostream>
using namespace std;

class rectangel{
     public:
          //下面这个是构造函数，它没有返回值，也就不用反回值类型
          rectangel(int l,int w){length = l;width = w;}
          //构造函数可以有多个，以参数不同为区分
          rectangel(){cout<<"这里调用无参的构造参数n";}

          //下面定义析构函数，析构函数也是没有返回值，且只能有一个没有参数的，使用~ 与构造函数进行区分
          ~rectangel(){cout<<"对象使用完毕，该说bye-bye啦！n";}

          int area(){return length*width;}
          int getLength(){return length;}
          int getWitdth(){return width;}
     private: //私有成员属性不能直接访问，要通过一个公有(public)的方法来返回
          int length;
          int width;
};

int main(){
     rectangel a(3,4);
     cout<<"这个长方形a的长为"<<a.getLength()<<endl;
     cout<<"这个长方形a的宽为"<<a.getWitdth()<<endl;
     cout<<"这个长方形a的面积为"<<a.area()<<endl;

     //这里创建另外一个对象，以无参的形式创建 ,这时它的长和宽将不确定
     rectangel b;
     cout<<"这个长方形b的长为"<<b.getLength()<<endl;
     cout<<"这个长方形b的宽为"<<b.getWitdth()<<endl;
     cout<<"这个长方形b的面积为"<<b.area()<<endl;
     return 0;

}
```

四、指针的常见错误
``` cpp
#include<iostream>
using namespace std;

int main(){
     int *p = new int;
     *p = 3;
     cout<<"初始化p时所指向的内存地址为："<<p<<endl;
     cout<<"将3赋给p的地址后，指针p读取的值为："<<*p<<endl;
     delete p;//将p指针删除，但是并没有将其指向空。删除掉指针其实并没有删除掉该指针
               //而是告诉编译器该指针所指向的这块内存区域我不用了，可以被别的变量所使用
               //而此时该指针所保存的地址在没有指向空之前还是原来指向的地址
     cout<<"删除指针后p所保存的内存地址为："<<p<<endl;
     long *p1 =new long;//这里重新开辟一块新的内存区域，也就是刚才释放掉的内存区域
}
```

```  cpp
将delete应用于指针时,它指向的内存将被释放,如果再一次对该指针使用delete,程序将崩溃,因为删除指针后要将其值设置为0或者NULL(即空指针),将delete用于空指针是安全的.

#include <iostream>
using namespace std;
int main()
{
    int* p = 0;//初始化时应该将指针赋值为空指针
    p = new int;
    *p = 9;
    cout<<"*p = "<<*p<<endl;
    delete p;
    p = 0;//or p = NULL;
    delete p;//delete p again,不会发生错误

    int n;//为了不让屏幕一闪而消失
    cin>>n;
    return 0;
}
```


如果delete p后未将p赋值为空指针的话,将会出现迷途指针,这个错误在C++中最难发现,最难解决的问题.迷途指针是指将delete用于指针p后,释放它所指向的内存,但是没有把指针赋值为空指针,这时候p所指向的内存可能供其他程序使用,一旦再一次调用p,因为p所指向的依然是那一块内存,那块内存又被其他程序已经用来,这个导致的结果不可预料,所以在C++中这个迷途指针才是麻烦.

就好比,我要订一个快餐,在手机上设置了快捷键2,按2就可以拨打400-517-517,快餐电话订快餐,但是后来这个电话号码有其他用途了,例如被设置为投诉电话,或者某人的家庭电话,又或者这个电话变成空号了,我按2快捷键的时候或者拨打到别人家里,别人公司,或者是空号,如果危险的话这个电话就被恐怖分子设置为启动炸弹的电话,一旦有电话打入,某地方的炸弹立刻爆炸,非常危险.所以在400-517-517这个号码变更好,我要把快捷键设置为空,等以后想用时再设置.

所以呢,delete p之后要将p赋值为空指针

五、常量指针
``` cpp
#include<iostream>
using namespace std;

class A {
public:
     int get(){return this->i;}
     void set(int x){this->i=x;}
private:
     int i;
};

int main(){
     int a = 3;
     int *const p = &a;//常量指针，指针所指向的内存地址不能改变，但是内存地址上的值可以改变,定义的时候必须要初始化
     a = 4;

     A *const p1 = new A;//指向A对象的一个常量指针
     //指针为常量，不可改变，但是指针所指的变量或者对象是可以改变的
     //p1 = p1+1;
     p1->set(11);
     cout<<p1->get();

     return 0;

}
```

六、指针与引用
``` cpp
#include<iostream>
using namespace std;

void swap(int &m,int &n){ //这里的参数是地址
     cout<<"交换前m的值为"<<m<<" n的值为:"<<n<<endl;
     int c;
     c=m;
     m=n;
     n=c;
     cout<<"交换后m的值为"<<m<<" n的值为:"<<n<<endl;

}

int func(int a,float *b,int *c){
     if(a>200||a<=0){
          a=0;
     }else{
          *b = a*a*3.14;
          *c = a*a;
          a = 1;
     }
     return a;

}

//使用按址传递的方式
int func2(int a,float &b,int &c){
     if(a>200||a<=0){
          a=0;
     }else{
          b = a*a*3.14;
          c = a*a;
          a = 1;
     }
     return a;

}

int main(){
     int num = 0;
     int &mum = num;//引用即别名，这里是将num取了另外一个名字叫mum
     num = 99;
     cout<<"mum的值为："<<mum<<endl;
     mum = 100;
     cout<<"num的值为:"<<num<<endl;
     cout<<"&num:"<<&num<<endl;
     cout<<"&mum:"<<&mum<<endl;//它们的内存地址是一样的
     cout<<"###################n";
     int b = 999;
     mum = b; //其实只是将b的值赋给了mum与num
     cout<<"将b赋给mun后的内存地址情况"<<endl;
     cout<<"&num:"<<&num<<":"<<num<<endl;
     cout<<"&mum:"<<&mum<<":"<<mum<<endl;
     cout<<"&b  :"<<&b<<":"<<b<<endl;
     cout<<"改变num的值后内存与值的情况"<<endl;
     num = 10;
     cout<<"&num:"<<&num<<":"<<num<<endl;
     cout<<"&mum:"<<&mum<<":"<<mum<<endl;
     cout<<"&b  :"<<&b<<":"<<b<<endl; //b的值是不受影响的

     cout<<"*******************n";
     int m = 3;
     int n = 4;
     cout<<"主程序中m的值为："<<m<<" n的值为："<<n<<endl;
     swap(m,n);//这里传的是m与n的地址
     cout<<"swap后m的值为："<<m<<" n的值为："<<n<<endl;

     cout<<"*******************n";
     cout<<"请输入一个数，将会求出圆的面积与正方形的面积n";
     int r,br;
     float cr;
     int check;
     cin>>r;
     check = func(r,&cr,&br);//这样可以通过一个函数来改变三个值 cr br a
     if(check){
          cout<<"圆的面积为："<<cr<<"正方形的面积为:"<<br<<endl;
     }else{
          cout<<"您输入的数字有误n";
          cout<<r<<"   "<<cr<<"   "<<br<<endl;
     }

     cout<<"请再次输入一个数，将会求出圆的面积与正方形的面积n";
     int x,z;
     float y;
     cin>>x;
     check = func2(x,y,z);//按址传递
     if(check){
          cout<<"圆的面积为："<<y<<"正方形的面积为:"<<z<<endl;
     }else{
          cout<<"您输入的数字有误n";
     }
     /*
     总的来说，使用按址传递的方法代码上看起来再简洁一些，指针容易写错
     */

     return 0;

}

```
