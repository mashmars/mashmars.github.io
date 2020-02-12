---
title: sqlalcheym-orm
date: 2020-02-12 17:04:07
tags:
    - python
    - sqlalcheym
categories: python
---

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String

engine = create_engine('mysql+pymysql://root:@localhost/python')
Base = declarative_base()

class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(20))
    fullname = Column(String(20))
    nickname = Column(String(20))

    def __repr__(self):
        return "<User(name='%s', fullname='%s', nickname='%s')>" % \
            (self.name, self.fullname, self.nickname)


``` print(User.__table__) ```

### 创建相关表
```  Base.metadata.create_all(engine) ```

### 创建映射类的实例
``` ed_user = User(name='mash', fullname="mashuai", nickname="mash")
print(ed_user.fullname) ```
### 创建会话 与数据库对话
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine)
session = Session()
### or 如果没有定义engine
``` Session = sessionmaker()
Session.configure(bind=engine) ```
#### 添加或更新对象
``` session.add(ed_user) # 尚未发出sql 如果此时查询数据库，将先刷新所有挂起的（但还是没有入库） 然后然后 如下
our_user = session.query(User).filter_by(name='mash').first() ```
``` print(our_user)
 执行查询将相当于执行如果语句， orm有数据了，但数据库里尚未插入 ```
``` BEGIN (implicit)
INSERT INTO users (name, fullname, nickname) VALUES (?, ?, ?)
('ed', 'Ed Jones', 'edsnickname')
SELECT users.id AS users_id,
        users.name AS users_name,
        users.fullname AS users_fullname,
        users.nickname AS users_nickname
FROM users
WHERE users.name = ?
 LIMIT ? OFFSET ?
('ed', 1, 0) ```
``` print(ed_user is our_user) ``` # True
#### 可以增加多个user对象
``` session.add_all([
    User(name="mash1", fullname="mashuai1", nickname="mash1"),
    User(name="mash2", fullname="mashuai2", nickname="mash2"),
    User(name="mash3", fullname="mashuai3", nickname="mash3"),
]) ```
##### 入库
```  session.commit() ```
### 查询
``` for row in session.query(User).order_by(User.id):
    print(row) ```
#### filter_by 使用关键字参数 .filter_by(name="mash")
#### filter 使用sql表达式语言构造
``` filter(User.name == 'mash')
filter(User.name != 'mash')
filter(User.name.like('%mash%'))  
filter(User.name.ilike('%ed%')) # 不区分大小写
filter(User.name.in_([]))    # in
filter(`User.name.in_([]))   # not in 
filter(User.name == None)    # is null
filter(User.name != None)    # is not null
filter(and_(User.name=="", User.fullname="")) => filter(User.name == '', User.fullname=="") \
=> filter(User.name == '').filter(User.fullname='')  # and 
filter(or_(User.name == '', User.name == ''))
```

### 返回列表和标量
``` all()
session.query(User).filter(...).order_by(User.id).all()
first()
one() \ 
# 完全获取所有行，如果结果不存在一个对象标识或复合航则会引发错误，多行raise MultipleResultsFound
# 找不到行 raise NoResultFound
one_or_none() # 没找到不报错， 只返回None ```
``` query = session.query(User).filter_by(name='mash')
print(query.scalar()) ```
### 使用文本sql
from sqlalchemy import text 
``` for user in session.query(User).filter(text('id<224')) \
    .order_by(text('id')).all():
    print(user.name) ```
``` user = session.query(User).filter(text("id<:id and name=:name")). \
    params(id = 224, name='mash').order_by(User.id).one()
print(user) ```
``` for user in session.query(User).from_statement(text('select * from users where name=:name')) \
    .params(name='mash').all():
    print(user) ```

### 聚合
from sqlalchemy import func
``` count = session.query(User).count()
print(count) ```
``` users = session.query(func.count(User.name), User.name).group_by(User.name).all()
print(users) ```
```  session.query(func.count(User.id)).scalar() ```

### 建立关系 relationship

from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

class Address(Base):
    __tablename__ = 'addresses'
    id = Column(Integer, primary_key=True)
    email_address = Column(String(20), nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'))

    user = relationship('User', back_populates="addresses")

    def __repr__(self):
        return "<Address(email_address='%s')>" % self.email_address

User.addresses = relationship('Address', order_by=Address.id, back_populates="user")

```  Base.metadata.create_all(engine) ```

### 使用相关对象
``` jack = User(name='jack', fullname="jack bean", nickname="giffdd")
 print(jack.addresses) ```
``` jack.addresses = [
    Address(email_address='jack@google.com'),
    Address(email_address='j25@yahoo.com')
] ```

``` print(jack.addresses[1])
 print(jack.addresses[1].user) ```

#### 保存
``` session.add(jack)
session.commit() ```

``` jack = session.query(User).filter_by(name='jack').one()
print(jack)
print(jack.addresses) ```

### 使用连接查询
``` for u, a in session.query(User, Address). \
                    filter(User.id == Address.user_id). \
                    filter(Address.email_address == 'jack@google.com'). \
                    all():
    print(u)
    print(a) ```
``` sql = session.query(User).join(Address). \
    filter(Address.email_address == 'jack@google.com')

print(sql) => select from users inner join addresses on ... ```

``` query.join(Address, User.id==Address.user_id)    # explicit condition
query.join(User.addresses)                       # specify relationship from left to right
query.join(Address, User.addresses)              # same, with explicit target
query.join('addresses')                          # same, using a string ```

``` sql = session.query(User).outerjoin(Address)
print(sql) => select from users left outer join addresses on ...  ```

### Query 如果存在多个实体，请从中选择？
```这个 Query.join() 方法意志 通常从最左边的项联接 
在实体列表中，如果省略了ON子句，或者ON子句是纯SQL表达式。要控制联接列表中的第一个实体，
请使用 Query.select_from() 方法： ```

``` query = session.query(User, Address).select_from(Address).join(User)
print(query) => select from addresses inner join  users on ... ```

### 减少查询 预先加载方式之一
from sqlalchemy.orm import joinedload

``` jack = session.query(User).options(joinedload(User.addresses)).filter_by(name='jack').one() ```

### 删除
``` jack = session.query(User).filter_by(name='jack').one_or_none()
session.delete(jack)
session.commit() ```

#### 配置关联删除
``` # User
addresses = relationship("Address", back_populates='user',cascade="all, delete, delete-orphan") ```

### 建立多对多 （需要创建中间表)
from sqlalchemy import Table, Text
post_keywords = Table('post_keywords', Base.metadata, 
    Column('post_id', ForeignKey('posts.id'), primary_key=True),
    Column('keyword_id', ForeignKey('keywords.id'), primary_key=True)
)
class BlogPost(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    handline = Column(String(100), nullable=False)
    body = Column(Text)

    keywords = relationship('Keyword', secondary=post_keywords, back_populates='posts')

    def __init__(self, handline, body, author):
        self.author = author
        self.handline = handline
        self.body = body
    
    def __repr__(self):
        return 'BlogPost(%r, %r, %r)' % (self.handline, self.body, self.author)

class Keyword(Base):
    __tablename__ = 'keywords'
    id = Column(Integer, primary_key=True)
    keyword = Column(String(50), nullable=False, unique=True)
    posts = relationship('BlogPost', secondary=post_keywords, back_populates='keywords')

    def __init__(self, keyword):
        self.keyword = keyword

``` 我们也想要我们的 BlogPost 类有一个 author 字段。
我们将把它添加为另一个双向关系，
除了一个问题，我们将有一个单一的用户可能有很多博客文章。
当我们进入 User.posts ，我们希望能够进一步筛选结果，以便不加载整个集合。
为此，我们使用 relationship() 调用 lazy='dynamic' ，用于配置替代项 装载机策略 在属性上： ```

BlogPost.author = relationship(User, back_populates='posts')
User.posts = relationship(BlogPost, back_populates='author', lazy='dynamic')

#### 创建表
``` Base.metadata.create_all(engine) ```

#### use
``` wendy = session.query(User).filter_by(name='jack').one()
post = BlogPost("jacks blog post", 'this is body', wendy)
session.add(post)
post.keywords.append(Keyword('jack'))
post.keywords.append(Keyword('firstpost'))

session.commit() ```

#### 查询
``` session.query(BlogPost).filter(BlogPost.keywords.any(keyword='firstword')).all() ```
