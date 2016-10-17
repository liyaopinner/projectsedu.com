---
title: sqlalchemy批量插入的坑
date: 2016-10-14 23:20:42
tags: python
---


默认提供的bulk_save_objects是在一次事务中提交多次save

需求：一个sql语句插入多个实体
直接上代码

models.py

    from sqlalchemy import (Column, String, DateTime, UnicodeText, BigInteger, Boolean, Integer, Text, Float)
    from sqlalchemy import TypeDecorator
    from sqlalchemy.ext.declarative import declarative_base
    
    Base = declarative_base()
    class People(Base):
        """
        people类
        """
        __tablename__ = 'people'
    
        id = Column(Integer, primary_key=True, unique=True, index=True, autoincrement=False)
        name = Column(String(100))
        age = Column(Integer, default=0)
        add_time = Column(DateTime, default=datetime.now, nullable=False)
    
    #注意 sqlalchemy字段默认可以为空，和django model相反

views.py

    value_list = []
    dict_list = []
    
    people1 = People()
    people2 = People()
    
    value_list.append(people1)
    value_list.append(people2)
    
    for item in value_list:
        item_dict = item.__dict__
        if "_sa_instance_state" in item_dict:
            del item_dict["_sa_instance_state"]
            dict_list.append(item_dict)
    
    
    engine = create_engine('mssql+pymssql://user:password@127.0.0.1:1433/data')
    DBSession = sessionmaker(bind=engine)
    session = TWDBSession()
    session.execute(timeline.__table__.insert(dict_list))
    session.commit()


    这里有个巨大的坑，如果people中任何一个实体没有某个字段，则即使其他实体有该字段也无法将该值插入到数据库中， 如下代码：
    
    people1 = People()
    
    people1.name = "bobby1"
    
    people2 = People()
    
    people2.name = "bobby1"
    people2.age = 3
    
    #注意people1没有设置age字段， 则在批量插入的时候， people1和people2都在数据库中都没有age字段。
    解决办法：  
        加上people1.age = None
        
    具体原因还没有看源码，以后有时间再研究