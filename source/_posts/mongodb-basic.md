---
title: mongodb-basic
date: 2019-06-03 17:53:16
tags: mongodb
categories: mongodb
---
#### 查询数据库
```
show database
show dbs
```
#### 创建数据库
If a database does not exist, MongoDB creates the database when you first store data for that database
```
use myDatabase
```
#### 查询当前所在数据库
```
db
```
#### 创建表
Collections are analogous to tables in relational databases
If a collection does not exist, MongoDB creates the collection when you first store data for that collection
```
db.createCollection('myTable')
db.myTable.insertOne({ ... })
```
#### 查询表
```
show tables
```
#### mongo shell
```
mongo # connect to default port 27017
mongo --port 28015 # connect to prot 28015
mongo "mongodb://mongodb.example.com:27017" # connect specify the hostname and port
mongo --host mongodb.example.com:27017 # the same to above 
mongo --host mongodb.example.com --port 27017 # 
```
#### mongodb instance with authentication
To connect to a mongodb instance requires authentication
You can specify the username, authentication database, and optionally the password in the connection string. For example, to connect and authenticate to a remote MongoDB instance as user alice:
```
mongo "mongodb://alice@mongodb.example.com:27017?authSource=admin"
```
You can use the --username < user> and --password, --authenticationDatabase < db> command-line options. For example, to connect and authenticate to a remote MongoDB instance as user alice:
```
mongo --username alice --password --authenticationDatabase admin --host mongodb.example.com --port 27017
```
#### mongodb curd operations
##### create operation
```
db.collection.insertOne()
db.inventory.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)
db.inventory.find({item: 'canvas'}).pretty()
db.collection.insertMany()
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
db.inventory.insert() # Inserts a document or documents into a collection.
db.inventory.insert(
    <document or array of documents>,
    {
        writeConcern: <document>,
        ordered: <boolean>
    }
)
```
##### query documents
###### 基本查询
``` 
db.inventory.find({}) => select * from inventory
db.inventory.find({status: 'D'}) => select * from inventory where status = 'D'
```
###### 使用查询操作符指定条件
格式为：{ < field1 >:{< operator1 >:< value>}, ... }
```
db.inventroy.find( { status: {$in: ['A', 'D' ]} }) => select * from inventory where status in ('A', 'D')
```
###### and 条件查询
```
db.inventory.find({ status: 'A', qty:{ $lt:30}}) => select * from inventory where status = 'A' and qty < 30
```
###### or 条件查询
```
db.inventory.find({ $or: [ {status:'A'}, {qty:{$lt:30}}]}) => select * from inventory where status = 'A' or qty < 30
```
###### and or 联合查询
```
db.inventory.find({ status:'A', $or:[ {qty:{$le:30}}, {item:/^p/}]}) => select * from inventory where status = 'A' and ( qty < 30 or item like 'p%')
```
###### 嵌入/嵌套文章查询
For example, the following query selects all documents where the field size equals the document { h: 14, w: 21, uom: "cm" }:
Equality matches on the whole embedded document require an exact match of the specified <value> document, including the field order.
```
db.inventory.find({ size: {h:14, w:21, uom:'cm'}})
```
###### 字段嵌套查询
当使用.dot查询的时候，一定要用引号引field字段
The following example selects all documents where the field uom nested in the size field equals "in":
```
db.inventory.find({'size.uom':'in'})
```
###### 字段嵌套查询 表达式查询
查询size.h < 15 的表达式
```
db.inventory.find({ 'size.h': {$lt:15}})
```
查询size.h < 15 and size.uom = 'in' and status = 'A'
note: size.带引号，status不用带
```
db.inventory.find({'size.h':{$lt:15},'size.uom':'in', status:'A'})
```
###### 数组查询
匹配一个数组查询，精确相当 ===
查询文档里的tags字段的值是一个数组，且该数组必须有两个元素red、blank
```
db.inventory.find({tags:['red','blank']})
```
数组查询 只要同时含有两个元素red、blank，且顺序无需一致以及不考虑是否存在其他元素
```
db.inventroy.find({tags:{$all:['red','blank']}})
```
数组查询 查询tags是一个数组，且里面有'red'元素
```
db.inventroy.find({tags:"red"})
```
the following operation queries for all documents where the array dim_cm contains at least one element whose value is greater than 25
```
db.inventory.find({dim_cm:{$gt:25}})
```
The following example queries for documents where the dim_cm array contains elements that in some combination satisfy the query conditions; e.g., one element can satisfy the greater than 15 condition and another element can satisfy the less than 20 condition, or a single element can satisfy both:
```
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )
```
Query for an Array Element that Meets Multiple Criteria
Use $elemMatch operator to specify multiple criteria on the elements of an array such that at least one array element satisfies all the specified criteria.

The following example queries for documents where the dim_cm array contains at least one element that is both greater than ($gt) 22 and less than ($lt) 30:
```
db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )
```
Query for an Element by the Array Index Position
Using dot notation, you can specify query conditions for an element at a particular index or position of the array. The array uses zero-based indexing.
The following example queries for all documents where the second element in the array dim_cm is greater than 25:
```
db.inventory.find( { "dim_cm.1": { $gt: 25 } } )
```
Query an Array by Array Length
Use the $size operator to query for arrays by number of elements. For example, the following selects documents where the array tags has 3 elements.
```
db.inventory.find( { "tags": { $size: 3 } } )
```
#### update delete
#### sql to mongodb mapping
|sql|mongodb|
|:---:|:---:|
|database|database|
|table|collection|
|row|document|
|column|field|
|index|index|
|table joins|$lookup|
|primary key|primary key|
|create table people(int,user_id,age)|db.people.insertOne({user_id:'123', age:22})|
|alter table perople add join_date datetime|db.people.updateMany({},{$set,{join_date: new Date()}})|
|alter table people drop column join_date|db.people.updateMany({},{$unset:{"join_date":""}})|
|create index idx_user_id on people(user_id)|db.people.createIndex({user_id:1})|
|create index idx_user_id_asc_age_desc on people(user_id,age desc)|db.createIndex({user_id:1, age:-1})|
|drop table people| db.people.drop()|
|insert into people values(1,'a',22)|db.people.insertOne({user_id: 'a', age: 22})|
|select * from people|db.people.find()|
|select id,user_id,age from people|db.people.find({},{user_id:1,age:1})|
|select user_id,age from people|db.people.find({},{_id:0,user_id:1,age:1})|
|select * from people where age=22|db.people.find({status:22})|
|select * from people where age <> 22 |db.people.find({status:{$ne:22}})|
|select * from people where user_id=2 and age=22|db.people.find({user_id:2,age:22})|
|select *from people where user_id=2  or age = 22 |db.people.find({$or:[{user_id:2,age:22}]})|
|select * from people where age>22 |db.people.find({age:{$gt:22}})|
|select * from people where age>22 and age < 24 |db.people.find({age:{$gt:22,$lt:24}})|
|select * from people where user_id like "%bc%"|db.people.find({user_id: /bc/}) or <br> db.people.find({user_id:{$regex:/bc/}})|
|select * from people where user_id like "bc%"|db.people.find({user_id: /^bc/}) or <br> db.people.find({user_id:{$regex:/^bc/}})|
|select * from people where age=22 order by user_id asc|db.people.find({age:22}).sort({user_id:1}) 1升序 -1倒序|
|select count(*) from people |db.people.count() <br> db.people.find().count()|
|select count(user_id) from people |db.people.count({user_id:{$exists:true}})<br>db.people.find({user_id:{$existx:true}}).count()|
|select count(*) from people where age>22|db.people.count({age:{$gt:22}}) <br> db.people.find({age:{$gt:22}}).count()|
|select distinct(user_id) from people|db.people.aggregate([{$group:{_id:"user_id"}}]) <br>db.people.distinct("user_id")|
|select * from people limit 1|db.people.findOne() <br> db.people.find().limit(1)|
|update people set age=23 where user_id >2 |db.people.updateMany({user_id:{$gt:2}},{$set:{age:23}})|
|update peopel set age=age+3 where user_id = 2 |db.people.updateMany({use_id:2},{$inc:{age:3}})|
|delete from people where age=23|db.people.deleteMany({age:23})|
|delete from people| db.people.deleteMany({})|









