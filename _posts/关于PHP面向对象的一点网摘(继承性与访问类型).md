title: 关于PHP面向对象的一点网摘(继承性与访问类型)
id: 205
permalink: 205
categories:
  - PHP
date: 2011-12-18 17:32:20
tags:
---

**类的继承**
继承作为面向对象的三个重要特性的一个方面，在面向对象的领域有着极其重要的作用，好像没听说哪个面向对象的语言不支持继承。继承是PHP5 面向对象程序设计的重要特性之 一，它是指建立一个新的派生类，从一个或多个先前定义的类中继承数据和函数，而且可以 重新定义或加进新数据和函数，从而建立了类的层次或等级。说的简单点就是，继承性是子类自动共享父类的数据结构和方法的机制，这是类之间的一种关系。在定义和实现一个类的 时候，可以在一个已经存在的类的基础之上来进行，把这个已经存在的类所定义的内容作为 自己的内容，并加入若干新的内容。比如你现在已经有一个“人”这个类了，这个类里面有 两个成员属性“姓名和年龄”以及还有两个成员方法“说话的方法和走路的方法”，如果现在 程序需要一个学生的类，因为学生的也是人，所以学生也有成员属性“姓名和年龄”以及成 员方法“说话的方法和走路的方法”，这个时候你就可以让学生类来继承人这个类，继承之后， 学生类就会把人类里面的所有的属性都继承过来，就不用你再去重新声明一遍这些成员属性 和方法了，因为学生类里面还有所在学校的属性和学习的方法，所以在你做的学生类里面有 继承自人类里面的属性和方法之外在加上学生特有的“所在学校属性”和“学习的方法”, 这样一个学生类就声明完成了，继承我们也可以叫做“扩展”，从上面我们就可以看出，学生 类对人类进行了扩展，在人类里原有两个属性和两个方法的基础上加上一个属性和一个方法 扩展出来一个新的学生类。 通过继承机制，可以利用已有的数据类型来定义新的数据类型。所定义的新的数据类型 不仅拥有新定义的成员，而且还同时拥有旧的成员。我们称已存在的用来派生新类的类为基 类，又称为父类以及超类。由已存在的类派生出的新类称为派生类，又称为子类。 在软件开发中，类的继承性使所建立的软件具有开放性、可扩充性，这是信息组织与分 类的行之有效的方法，它简化了对象、类的创建工作量，增加了代码的可重性。采用继承性， 提供了类的规范的等级结构。通过类的继承关系，使公共的特性能够共享，提高了软件的重
用性。
在C++语言中，一个派生类可以从一个基类派生，也可以从多个基类派生。从一个基类派生的继承称为单继承；从多个基类派生的继承称为多继承。<span style="color: #ff0000;">但是在PHP 和Java 语言里面没有多继承，只有单继承，也就是说，一个类只能直接从 一个类中继承数据，这就是我们所说的单继承。 也就是说一个子类只能有一个父类，但一个父类可以有多个子类</span>
例如：
下面是“人”类的抽象 <!-- more -->
代码片段
``` php 
class Person{ //定义一个“人”类作为父类
//下面是人的成员属性
var $name;
var $sex;
var $age;
//定义一个构造方法参数为属性姓名$name、性别$sex和年龄$age进行赋值
function __construct($name, $sex, $age){
$this->name=$name;
$this->sex=$sex;
$this->age=$age;
}
//这个人可以说话的方法, 说出自己的属性
function say() {
echo "我的名子叫：".$this->name." 性别：".$this->sex." 我的年龄是：".$this->age."<br>";
}
}
```
 

下面我们做一个“学生类”，如果不是用继承如下：代码片段
<div>
``` php 
class Student{ //定义一个“人”类做为父类
//下面是人的成员属性
var $name;
var $sex;
var $age;
var $school; //新添加的学生所在学校的属性
//定义一个构造方法参数为属性姓名$name、性别$sex和年龄$age进行赋值
function __construct($name=””, $sex=””, $age=””, $school=””){
$this->name=$name;
$this->sex=$sex;
$this->age=$age;
$this->school=$school;
}
//这个人可以说话的方法, 说出自己的属性
function say() {
echo "我的名子叫：".$this->name." 性别：".$this->sex." 我的年龄是：".$this->age."<br>";
}
//这个学生学习的方法
function study() {
echo "我的名子叫：".$this->name." 我正在”.$this->school.”学习<br>";
}
}
//定义一个子类“学生类“使用”extends”关键字来继承”人”类
class Student extends Person{
var $school; //学生所在学校的属性 只是新添加上去
//这个学生学习的方法
function study() {
echo "我的名子叫：".$this->name." 我正在”.$this->school.”学习<br>";
}
}
```
 

通过上面“Student”类的定义，Student 类通过使用“extends”这个关键字把Person 类里的所有成员属性和成员方法都继承过来了，并扩展了一个所在学校成员属性“school”，和一个学习方法“study()”。现在子类“Student”里面和使用这个类实例出来的对象都具有如下的属性和方法：
学生类“Student”里面的成员属性有：
姓名：name;
年龄：age;
性别：sex;
学校：school;
学生类“Student”里面的成员方法有：
说话方法：say();
学习方法：study();
通过上面类继承的使用简化了对象、类的创建工作量，增加了代码的可重性。但是从上面这一个例子上中“可重用性”以及其它的继承性所带来的影响，我们看的还不是特别的明显，你扩展的去想一下，人有无数个岗位，比如上面的学生还有老师、工程师、医生、工人 等，很多很多，如果每个类都定义“人”都共同具有的属性和方法，想一想会有很大的工作 量，这些属性和方法都可以从“Person”人类里面继承过来。

<strong>重载新的方法</strong> (只是在子类里重新定义父类中的函数)
在学习PHP 这种语言中你会发现，PHP 中的方法是不能重载的，所谓的方法重载就是定义相同的方法名，通过“参数的个数”不同或“参数的类型”不同,来访问我们的相同方法 名的不同方法。但是因为PHP 是弱类型的语言，所以在方法的参数中本身就可以接收不同类 型的数据，又因为PHP 的方法可以接收不定个数的参数，所以通过传递不同个数的参数调用不相同方法名的不同方法也是不成立的。所以在PHP 里面没有方法重载。不能重载也就是在 你的项目中不能定义相同方法名的方法。另外，因为PHP 没有名子空间的概念，在同一个页 面和被包含的页面中不能定义相同名称的方法，也不能定义和PHP 给我提供的方法重名，当然在同一个类中也不能定义相同名称的方法。我们这里所指的重载新的方法所指的是什么呢？其实我们所说的重载新的方法就是子类覆盖父类的已有的方法，那为什么要这么做呢？父类的方法不是可以继承过来直接用吗？但 有一些情况是我们必须要覆盖的，比如说我们前面提到过的例子里面，“Person”这个人类里 面有一个“说话”的方法，所有继承“Person”类的子类都是可以“说话”的，我们“Student” 类就是“Person”类的子类，所以“Student”的实例就可以“说话”了，但是人类里面“说 话”的方法里面说出的是“Person”类里面的属性，而“Student”类对“Person”类进行了扩 展，又扩展出了几个新的属性，如果使用继承过来的“say()”说话方法的话，只能说出从 “Person”类继承过来的那些属性，那么新扩展的那些属性使用这个继承过来的“say()”的 方法就说不出来了，那有的人就问了，我在“Student”这个子类中再定义一个新的方法用于 说话，说出子类里面所有的属性不就行了吗？一定不要这么做，从抽象的角度来讲，一个“学 生”不能有两种“说话”的方法，就算你定义了两个不同的说话的方法，可以实现你想要的 功能，被继承过来的那个“说话“方法可能没有机会用到了，而且是继承过来的你也删不掉。 这个时候我们就要用到覆盖了。虽然说在PHP 里面不能定义同名的方法，但是在父子关系的两个类中，我们可以在子类 中定义和父类同名的方法，这样就把父类中继承过来的方法覆盖掉了。 代码片段
``` php 
<?

class Person{ //定义一个“人”类做为父类
//下面是人的成员属性
var $name;
var $sex;
var $age;
//定义一个构造方法参数为属性姓名$name、性别$sex和年龄$age进行赋值
function __construct($name, $sex, $age){
$this->name=$name;
$this->sex=$sex;
$this->age=$age;
}

function say() {
echo "我的名子叫：".$this->name." 性别：".$this->sex." 我的年龄是：".$this->age."<br>";
}
}
class Student extends Person
{
var $school;
function study() {
echo "我的名子叫：".$this->name." 我正在”.$this->school.”学习<br>";
}
//这个学性可以说话的方法, 说出自己所有的属性，覆盖了父类的同名方法
function say() {
echo "我的名子叫：".$this->name." 性别：".$this->sex." 我的年龄是：".$this->age."我在".$this->school."上学.<br>";
}
}
?>
```

上面的例子，我们就在“Student”子类里覆盖了继承父类里面的“say()”的方法，通过覆盖我们就实现了对“方法”扩展。 但是，像这样做虽然解决了我们上面说的问题，但是在实际开发中，一个方法不可能就一条代码或是几条代码，比如说“Person”类里面“say()”方法有里面有100 条代码，如果我们想对这个方法覆盖保留原有的功能外加上一点点功能，就要把原有的100 条代码重写 一次，再加上扩展的几条代码，这还算是好的，而有的情况，父类中的方法是看不见原代码 的，这个时候你怎么去重写原有的代码呢？我们也有解决的办法，就是在子类这个方法中可 以调用到父类中被覆盖的方法，也就是把被覆盖的方法原有的功能拿过来再加上自己的一点 功能，可以通过两种方法实现在子类的方法中调用父类被覆盖的方法：一种是使用父类的“类名：：“来调用父类中被覆盖的方法；一种是使用“parent：：”的方试来调用父类中被覆盖的方法；
<div>
``` php 
class Student extends Person{
var $school;
function study() {
echo "我的名子叫：".$this->name." 我正在”.$this->school.”学习<br>";
}
//这个学性可以说话的方法, 说出自己所有的属性，覆盖了父类的同名方法
function say() {
//使用父类的“类名::“来调用父类中被覆盖的方法；
// Person::say();
//或者使用“parent：：”的方试来调用父类中被覆盖的方法；这种方法没法加上了类的函数
parent::say();
//加上一点自己的功能
echo “我的年龄是：".$this->age."我在".$this->school."上学.<br>";
}
}
```
 

现在用两种方式都可以访问到父类中被覆盖的方法，我们选那种方式最好呢？用户可能会发现自己写的代码访问了父类的变量和函数。如果子类非常精炼或者父类非常专业化的时候尤其是这样。不要用代码中父类文字上的名字，应该用特殊的名字parent，它指的就是子类在extends 声明中所指的父类的名字。这样做可以避免在多个地方使用父类的名字。如果继承树在实现的过程中要修改，只要简单地修改类中extends 声明的部分。同样，构造方法在子类中如果没有声明的话，也可以使用父类中的构造方法，如果子类中重新定义了一个构造方法也会覆盖掉父类中的构造方法，如果想使用新的构造方法为所有属性赋值也可以用同样的方式。
代码片段
<div>
``` php 
class Student extends Person{
var $school; //学生所在学校的属性
function __construct($name, $sex, $age, $school){
//使用父类中的方法为原有的属性赋值
parent::__construct($name, $sex, $age);
$this->school=$school;
}
//这个学生学习的方法
function study() {
echo "我的名子叫：".$this->name." 我正在”.$this->school.”学习<br>";
}
//这个人可以说话的方法, 说出自己的属性
function say() {
parent::say();
//加上一点自己的功能
echo “我的年龄是：".$this->age."我在".$this->school."上学.<br>";
```
 

</div>
<div>**访问类型**
类型的访问修饰符允许开发人员对类成员的访问进行限制，这是PHP5 的新特性，但却是OOP 语言的一个好的特性。而且大多数OOP 语言都已支持此特性。PHP5 支持如下3 种访问修饰符public (公有的、默认的)，private (私有的)和protected (受保护的)三种。 public 公有修饰符，类中的成员将没有访问限制，所有的外部成员都可以访问（读和写）这个类成员（包括成员属性和成员方法），在PHP5 之前的所有版本中，PHP 中类的成员都是public 的，而且在PHP5 中如果类的成员没有指定成员访问修饰符，将被视为public。 例：public $name;</div>
<div>public function say(){};
private 私有修改符，被定义为private 的成员，对于同一个类里的所有成员是可见的，即是没有访问限制；但对于该类的外部代码是不允许改变甚至读操作，对于该类的子类，也不能访问private 修饰的成员。例：private $var1 = ‘A'; //属性
private function getValue(){} //函数
protected 保护成员修饰符，被修饰为protected 的成员不能被该类的外部代码访问。但是对于该类的子类有访问权限，可以进行属性、方法的读及写操作,该子类的外部代码包括其的子类都不具有访问其属性和方法的权限。
例：protected $name;
protected function say(){};
                    private        protected            public
同一个类中             √             √                   √
类的子类中             √             √
所有的外部成员         √
代码片段
``` php 
<?php
/**
* Define MyClass
*/
class MyClass{
// Contructors must be public
public function __construct() { }
// Declare a public method
public function MyPublic() { }
// Declare a protected method
protected function MyProtected() { }
// Declare a private method
private function MyPrivate() { }
// This is public
function Foo() {
$this->MyPublic();
$this->MyProtected();
$this->MyPrivate();
}
}
$myclass = new MyClass;
$myclass->MyPublic(); //Works
$myclass->MyProtected(); // Fatal Error
$myclass->MyPrivate(); // Fatal Error
$myclass->Foo(); // Public, Protected and Private work
/**
* Define MyClass2
*/
class MyClass2 extends MyClass{
// This is public
function Foo2(){
$this->MyPublic();
$this->MyProtected();
$this->MyPrivate(); // Fatal Error
}
}
$myclass2 = new MyClass2;
$myclass2->MyPublic(); // Works
$myclass2->Foo2(); // Public and Protected work, not Private
?>
```
另外在子类覆盖父类的方法时也要注意一点，子类中方法的访问权限一定不能低于父类被覆盖方法的访问权限，也就是一定要高于或等于父类方法的访问权限。 例如，如果父类方法的访问权限是protected 那么子类中要覆盖的权限就要是protected
和public，如果父类的方法是public 那么子类中要覆盖的方法只能也是public，总之子类中的方法总是要高于或等于父类被覆盖方法的访问权限.
