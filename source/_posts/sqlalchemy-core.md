---
title: sqlalchemy-core
date: 2020-02-12 17:03:58
tags:
    - python
    - sqlalchemy
categories: python
---
```
from sqlalchemy import create_engine
from sqlalchemy import Table, Column, Integer, String, MetaData, ForeignKey

engine = create_engine('mysql+pymysql://root:@localhost/python')

metadata = MetaData()

users = Table('users', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(20)),
    Column('fullname', String(20))
)

addresses = Table('addresses', metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', None, ForeignKey('users.id')),
    Column('email', String(20), nullable=False, key='email')
)
```
### 创建表 删除表drop_all 创建删除单个表 users.create(engine)
```  metadata.create_all(engine)  ```

### 插入数据
``` ins = users.insert().values(name="mash", fullname='mashuai')
# print(str(ins))
# print(ins.compile().params)
## 执行
conn = engine.connect()
result = conn.execute(ins)

## or 这样插入数据
ins = users.insert()
conn = engine.connect()
conn.execute(ins, name="mash1", fullname="mashuai1") ```

### 执行多个语句
``` ins = users.insert()
conn = engine.connect()
conn.execute(ins, [
    {"name":"mash2", "fullname":"mashuai2"},
    {"name":"mash3", "fullname":"mashuai3"},
    {"name":"mash4", "fullname":"mashuai4"},  
]) ```

### select
from sqlalchemy.sql import select
``` s = select([users])
result = engine.connect().execute(s)
for row in result:
    print(row)
# print(result.fetchone())
## 还有另一种方法，其效用将在以后变得明显， 是使用Column直接作为键的对象：
for row in result:
    print("name:", row[users.c.name], ", fullname:", row[users.c.fullname])
```
#### 如果想要控制select的列
``` s = select([users.c.name])
result = engine.connect().execute(s)
for row in result:
    print(row) ```
####加条件
``` s = select([users]).where(users.c.id == 2)
result = engine.connect().execute(s)
print(result.fetchone()) ```
#### 算子   
``` print(users.c.id == addresses.c.user_id)
print(users.c.id == 2)
 ```
### 连词 and_ or_ not_
### 排序 及 函数 .order_by(users.c.name.desc())
``` from sqlalchemy import func, desc
stmt = select([users.c.name, func.count(users.c.id).label('count')]).group_by('id').order_by(desc('id'), desc('name'))
result = engine.connect().execute(stmt).fetchall()
for row in result:
    print(row)
    print(row['count'])
 ```
### 使用别名
```  u = users.alias('u') ```
### 连接 join() outerjoin()
``` print(users.join(addresses))
print(users.join(addresses,
                        addresses.c.email_address.like(users.c.name + '%')
                )
    )
s = select([users.c.fullname]).select_from(
    users.join(addresses, addresses.c.email.like(users.c.name + '%'))
)
result = engine.connect().execute(s).fetchall()
s = select([users.c.fullname]).select_from(users.outerjoin(addresses))
print(s)
 ```
### 绑定参数对象
from sqlalchemy.sql import bindparam
``` s = users.select(users.c.name == bindparam('username'))
result = engine.connect().execute(s, username='mash4').fetchall()
for row in result:
    print(row) ```

### 排序 分组 限制 偏移
``` s = select([users.c.name]).order_by(users.c.name.desc()).group_by(users.c.name) \
    .having(users.c.name != 'mash1').distinct().limit(1).offset(1)
 ```
### 插入 更新 删除
``` s = users.update().values(fullname="ssss").where(users.c.name=="mash4")
engine.connect().execute(s) ```
``` s = users.update().where(users.c.name == bindparam('oldname')).values(name=bindparam('newname'))
engine.connect().execute(s, [
    {'oldname':"mash1", 'newname':'mash11'},
    {'oldname':"mash2", 'newname':'mash22'},
]) ```

### 删除
``` engine.connect().execute(addresses.delete())  # delete from addresses 
engine.connect().execute(users.delete().where(users.c.name == 'mash4')) ```
### 获取影响行数
``` result = engine.connect().execute(addresses.delete()) 
result.rowcount
 ```


