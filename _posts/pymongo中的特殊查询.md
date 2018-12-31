title: pymongo中的特殊查询
comment: true
categories:
  - Python
date: 2018-12-27 23:53:38
tags: python
permalink: specialUse-in-pymongo

---
python操作mongo常用的增删改查操作。[Python操作MongoDB看这一篇就够了](https://juejin.im/post/5addbd0e518825671f2f62ee)

最近在整理一些在pymongo中相对使用较少的查询。

<!-- more -->

# 忽略大小写查询

可以使用正则(re)来查询

``` python 
import re
from pymongo import Connection

name = r"^%s$"%filename
re_name = re.compile(name,re.I)
conn = Connection(host, port)
fileinfos = conn.fileRecode.fileTask.find({'filename':re_name},projection={'_id': False})

```

也可以使用 `$regex` 表达式，上面的代码可以使用下面替换
``` python 
fileinfos = conn.fileRecode.fileTask.find({'filename':{'$regex':r"^%s$"%filename,'$options':"i"}},projection={'_id': False})

```

`$options` 目前支持四种，比较常用的是`i `

| 选项 | 含义 | 使用要求 |
| ------ | ------ | ------ |
|i       |大小写不敏感|     |
|m       |查询匹配中使用了锚，例如^（代表开头）和$（代表结尾），以及匹配\n后的字符串|
|x       |忽视所有空白字符  |要求$regex与$option合用
|s       |允许点字符（.）匹配所有的字符，包括换行符。|要求$regex与$option合用


# 字段的多层结构查询

比如有如下格式的数据
``` json
/* 1 */
{
    "_id" : ObjectId("5c24a27ddb1f5a209cf8799a"),
    "info" : {
        "age" : "30",
        "name" : "yangyanxing"
    },
    "id" : 1
}

/* 2 */
{
    "_id" : ObjectId("5c24a2acdb1f5a209cf8799b"),
    "info" : {
        "age" : "20",
        "name" : "fanjy"
    },
    "id" : 1
}

```
比如我要查询info下的name为yangyanxing的该如何查询呢？
使用`.`来连接查询，这里就是`info.name`
``` python 
>>> conn.yang.test.find_one({"info.name":"fanjy"})
{u'info': {u'age': u'20', u'name': u'fanjy'}, u'_id': ObjectId('5c24a2acdb1f5a209cf8799b'), u'id': 1}

```


# 查询指定时间段的数据

有时候在插入数据的时候会使用`datetime.datetime.now()` 来插入一个datatime格式的数据，比如要查询某段时间的数据该怎么查询呢？
可以使用`$gte`(大于等于)与`$lte`(小于等于)来查询，比如说要查询一年以来的数据，可以这样
``` python 
import datedate
from pymongo import Connection

start_check = datetime.datetime.now()-datetime.timedelta(days=365)
checkCon = {'addtime':{"$gte":start_check}}
conn = Connection(host, port)
fileinfos = conn.Documents.find(checkCon)

```


# 逻辑关系查询
一般的逻辑查询包括`与 或 非`
1. 关系`与` (and)
其实这个不要特殊的关键词，直接将要查询的内容依次写到查询条件中就是`与`的关系
``` python
>>> conn.yang.test.find_one({"info.name":"fanjy","id":1})
{u'info': {u'age': u'20', u'name': u'fanjy'}, u'_id': ObjectId('5c24a2acdb1f5a209cf8799b'), u'id': 1}
```

2. 关系`或` (or) 需要使用`$or`,比如要查询名字叫 yangyanxing 或者 地址在tongzhou的

``` python 
coon.yang.test.find({"$or":[{"name":"yangyanxing"},{"address":"tongzhou"}]})

```
`$or`的值为一个列表，列表中的每一项是一个字典，字典中是各种查询条件。

列表也可以用列表迭代，比如我要找到 taskid 为1,2,4,6,7,9,10,20,34的任务,可以使用
``` python 
conn.yang.test.find({'$or':[{"taskid":i} for i in l]})
```

3. `$in` 操作
``` python 
>>> for i in conn.yang.test.find({"info.name":{"$in":['yangyanxing','fanjy']}}):
...     print i
...
{u'info': {u'age': u'30', u'name': u'yangyanxing'}, u'_id': ObjectId('5c24a27ddb1f5a209cf8799a'), u'id': 1}
{u'info': {u'age': u'20', u'name': u'fanjy'}, u'_id': ObjectId('5c24a2acdb1f5a209cf8799b'), u'id': 1}
>>> for i in conn.yang.test.find({"info.name":{"$in":['yangyanxing','xxxxx']}}):
...     print i
...
{u'info': {u'age': u'30', u'name': u'yangyanxing'}, u'_id': ObjectId('5c24a27ddb1f5a209cf8799a'), u'id': 1}

```

参考文章
[3分钟掌握MongoDB中的regex几种用法](http://blog.51cto.com/suifu/2070686)







