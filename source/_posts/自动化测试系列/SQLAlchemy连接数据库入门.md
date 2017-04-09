---
title: "SQLAlchemy连接数据库入门"
date: 2017-04-02 21:55:58
tags: [Web-Test, automation]
categories: [automation]
---
自动化测试中常常需要访问数据库来获得所需数据，Python里访问数据库的Library有很多，例如MySQL ，SQLite， SQLAlchemy 等，运用最广泛的当属SQLAlchemy， 我们今天就以SQLAlchemy为例，讲下如何连接，使用，更新数据库。
<!--more-->
>SQLAlchemy is the Python SQL toolkit and Object Relational Mapper that gives application developers the full power and flexibility of SQL.
SQLAlchemy is most famous for its object-relational mapper (ORM), an optional component that provides the data mapper pattern, where classes can be mapped to the database in open ended, multiple ways - allowing the object model and database schema to develop in a cleanly decoupled way from the beginning.

如何安装：   
{% codeblock lang:python %}        
pip install SQLAlchemy
{% endcodeblock %}
如何使用：
{% codeblock lang:python %}    
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy import Column, DateTime, String, Integer, text, and_
from sqlalchemy.ext.declarative import declarative_base

# 1
base = declarative_base()
class ClassAudit(base):
    __tablename__ = 'ClassAudit'

    Class_id = Column(Integer)
    StudentMember_id = Column(Integer)
    Comment = Column(String(100))
    ...
# 2 
class DBHelper(object):
    def __init__(self):
        self.engine = create_engine("mssql+pymssql://用户名:密码@数据库ip/数据库名",
                                    encoding='latin1', echo=True)
        DBSession = sessionmaker(bind=self.engine)
        self.dbsession = DBSession()

    def query_status(self, class_id, message):
         query = self.dbsession.query(ClassAudit).filter(and_(ClassAudit.Class_id==class_id, ClassAudit.Comment.like('%'+ '%s' % message + '%' )))
         return query.all()

if __name__ == "__main__":
    db_helper = DBHelper()
    # Method 1
    txt = db_helper.query_status('1501287', 'test')
    for i in txt:
        print i.Comment

    # Method 2
    sql_string = """select * from teachers..classaudit where class_id='1501287' and comment like '%test%'"""
    conn = db_helper.engine.connect()
      txt = conn.execute(sql_string)
     for i in txt:
       print i.Comment
{% endcodeblock %}
根据上面的代码我们来逐步说明sqlalchemy的使用步骤：
1. 连接数据库
{% codeblock lang:python %}  
>>> from sqlalchemy import create_engine
>>> engine = create_engine('sqlite:///:memory:', echo=True)
{% endcodeblock %}
create_engine()用来初始化数据库连接，语法如下：
'数据库类型+数据库驱动名称://用户名:口令@机器地址:端口号/数据库名'
2.  声明mapping关系
参考上面代码的#1和#2之间的代码。 ClassAudit是数据库表名，这里我们用来来和数据库中真实存在的table对应。要对应的table可以是已经存在的，也可以是要新建立的，如果是新建立的，那么顺序执行下面的3步。否则略过。
3.  创建一个数据库表格并插入数据
#创建数据库
{% codeblock lang:python %}   
from sqlalchemy import Table, MetaData
 
engine = create_engine('sqlite:///:memory:')
metadata = MetaData()
 
user = Table('user', metadata,
    Column('user_id', Integer, primary_key=True),
    Column('user_name', String(16), nullable=False),
    Column('email_address', String(60), key='email'),
    Column('password', String(20), nullable=False)
)
 
metadata.create_all(engine)
 
#插入数据
ins = user.insert().values(name='kevin', fullname='Kevin cai')
conn = engine.connect()
result = conn.execute(ins)
{% endcodeblock %}
4.   创建一个session
{% codeblock lang:python %}   
DBSession = sessionmaker(bind=self.engine)
self.dbsession = DBSession()
{% endcodeblock %}
5.   增加或者更新数据库对象
当需要添加数据库记录时使用，也可以用第3步的方法来插入数据。
{% codeblock lang:python %}   
new_audit = ClassAudit(Class_id=1, =’123’,,,,)
self.dbsession.add(ClassAudit)
self.dbsession.commit()
self.dbsession.close()
{% endcodeblock %}
6.   查询
Query的关键字和函数用法如下：
{% codeblock lang:python %}   
AND:
from sqlalchemy import and_
session.query.filter(and_(User.name == 'kevin', User.fullname == kevin cai'))
LIKE:
session.query.filter(User.name.like('%test'))
all() :returns a list:
query = session.query(User).filter(User.name.like('%ed')).order_by(User.id)
query.all()
{% endcodeblock %}
以上就是sqlalchemy对数据库的操作，这个通常在有很多表格并且这些表格有联系的情况下使用，如果你的代码里只需要查询独立的几个表格，你完全可以不建立表格的对应关系，直接使用sql语句，做法为删除掉上面#1和#2之间的代码，并且参考main函数中Method2的代码即可。

SQLAlchemy是ORM技术的典型代表，希望大家都学会如何使用（Object-Relational Mapping，把关系数据库的表结构映射到对象上)。