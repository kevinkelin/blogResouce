title: mysql与mongodb命令对比
id: 1128
permalink: 1128
categories:
  - Python
date: 2014-09-29 01:01:51
tags:
  - sql
---

之前一直在使用mysql,最近开始接触mongodb，觉得还是有一些相似的地方，只是相应的命令不大一样，这里做下记录，对比记应该相对简单一些,mongodb使用的是python的接口

连接：
mysql: mysql -h localhost -u username -p
mongodb:con = pymongo.Connection('localhost',27017)
显示数据库
mysql:show databases;
mongodb:con.database_names()
使用某个数据库
mysql:use database;
mongodb:db = con['database'] or con.database 没有将会创建
显示所有的表
mysql:show tables;
mongodb:db.collection_names()
使用某个表(mongodb中称为colection)
mysql: use table;
mongodb:col = db.collection or db['collection'] 没有将会创建
遍历表中的内容
mysql: select * from table[where...]
mongodb: for i in col.find():print i
for i in col.find_one({'key':'value'}):print i 返回值是字典
向表中插入值
mysql: insert into table valuses(......)
mongodb:col.insert({'key':'value',...})
d = {'key':'value'}
col.insert(d)