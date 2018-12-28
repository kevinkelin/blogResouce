title: python中的list排序问题以及sort,sorted的使用
comment: true
categories:
  - Python
date: 2018-06-28 23:53:38
tags: python
permalink: python-list-sort

---

列表list排序是一种非常常见的操作，里面有很多的方法也有很多的坑，这里简单的记录一下。

<!-- more -->

1. sorted 方法

``` python
>>> l = ['1','2','A','z','a',1,2]
>>> sorted(l)
[1, 2, '1', '2', 'A', 'a', 'z']
>>> l
['1', '2', 'A', 'z', 'a', 1, 2]
>>> l2 = sorted(l)
>>> l2
[1, 2, '1', '2', 'A', 'a', 'z']
>>> id(l2)
38967136
>>> id(l)
38985520
```

sorted 方法传入一个列表，返回一个新的列表，并不影响原来列表的顺序，从上面的结果来看，数字排在了字符串之前，其实它是按照字符的ASCII的顺序排列的，那如果列表里是字符串呢？ 

``` python 
>>> l = ["yang","yan","xing","is","a","boy"]
>>> sorted(l)
['a', 'boy', 'is', 'xing', 'yan', 'yang']

```

明显它是按照第一个字符来排，第一个字符相同再按第二个排。

sorted() 方法还有另外一个参数，reverse，当它为True的时候，将会倒序排列。

``` python 
>>> sorted(l,reverse=True)
>>> ['yang', 'yan', 'xing', 'is', 'boy', 'a']

```


可否直接改变原list的结构，而不是生成一个新的list呢？ 当然可以  

2. sort方法

```
>>> l
['yang', 'yan', 'xing', 'is', 'a', 'boy']
>>> l.sort()
>>> l
['a', 'boy', 'is', 'xing', 'yan', 'yang']
>>>
```

此时原列表l的结构已经变了，且该函数没有返回值，所以如果写成`newL = l.sorr()` 并不能得到一个新的列表，newL为空，所以不要用错了

这种排序也太简单了，我可否按照我自已的方法来排序呢？当然可以

3. 自定义排序

需要提供一个排序的方法，该方法要有三种返回值，假如有两个值a1和a2，通过某种计算，如果返回大于0(True)的值，则a1排在后面，如果小于0(False)，a1排在前面，如果等于0，则a1和a2顺序不变，看代码  

``` python
#coding:gbk

a = [
    {'name':'jk', 'score':100, 'first':50, 'second':50, 'third':0}, 
    {'name':'zz', 'score':90, 'first':50, 'second':30, 'third': 10}, 
    {'name': 'yyx', 'score':90, 'first':40, 'second':30, 'third':20},
    {'name': 'zs', 'score':90, 'first':40, 'second':30, 'third':20},
]

def cmp(a1, a2):
    if a1['score'] != a2['score']:
        return a1['score'] - a2['score']
    elif a1['first'] != a2['first']:
        return a1['first'] - a2['first']
    elif a1['second'] != a2['second']:
        return a1['second'] - a2['second']
    else:
        return a1['third'] - a2['third']
        
a.sort(cmp)

for i in a:
    print i
```

这里引出另外一个问题，字典是无序的，想要排字典，其实是排一组字典
这里希望将上述的list排序，按照每条记录中的'score'排序。如果'score'字段的值相等，则按照'first'的值排序。如果'first'依旧相等，则按照'second'排序。如果'second'相等，则按照'third'字段的值来排序。
该排序的结果为
>{'second': 50, 'score': 100, 'name': 'jk', 'third': 0, 'first': 50}
{'second': 30, 'score': 90, 'name': 'zz', 'third': 10, 'first': 50}
{'second': 30, 'score': 90, 'name': 'yyx', 'third': 20, 'first': 40}
{'second': 30, 'score': 90, 'name': 'zs', 'third': 20, 'first': 40}

还可以用sorted方法

``` python 
>>> student_tuples = [('john', 'A', 15),('jane', 'B', 12),('dave', 'B', 10)]
# sort by age
>>> sorted(student_tuples, key=lambda student: student[2])
[('dave', 'B', 10), ('jane', 'B', 12), ('john', 'A', 15)]
```


参考文章
[python list 自定义排序](https://blog.csdn.net/nisxiya/article/details/49305857)
[Python之排序函数sort() 和 sorted()](https://www.jianshu.com/p/7be04a3f30cd)
[Python的排序：关于sort()与sorted()](https://blog.csdn.net/qq_15714857/article/details/50545265)




















