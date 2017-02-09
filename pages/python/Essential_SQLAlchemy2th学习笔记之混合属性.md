modifyAt:2016-12-18 00:47:24
title:Essential SQLAlchemy2th学习笔记之混合属性
createAt:2016-12-18 00:47:24
location:python/Essential_SQLAlchemy2th学习笔记之混合属性
author:xbynet

[官方文档](http://docs.sqlalchemy.org/en/latest/orm/extensions/hybrid.html)

```
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:///:memory:')

Session = sessionmaker(bind=engine)
from datetime import datetime

from sqlalchemy import Column, Integer, Numeric, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.ext.hybrid import hybrid_property, hybrid_method

Base = declarative_base()


class Cookie(Base):
    __tablename__ = 'cookies'

    cookie_id = Column(Integer, primary_key=True)
    cookie_name = Column(String(50), index=True)
    cookie_recipe_url = Column(String(255))
    cookie_sku = Column(String(55))
    quantity = Column(Integer())
    unit_cost = Column(Numeric(12, 2))
    
    @hybrid_property
    def inventory_value(self):
        return self.unit_cost * self.quantity
    
    @hybrid_method
    def bake_more(self, min_quantity):
        return self.quantity < min_quantity
        
    def __repr__(self):
        return "Cookie(cookie_name='{self.cookie_name}', " \
                       "cookie_recipe_url='{self.cookie_recipe_url}', " \
                       "cookie_sku='{self.cookie_sku}', " \
                       "quantity={self.quantity}, " \
                       "unit_cost={self.unit_cost})".format(self=self)
 

Base.metadata.create_all(engine)
```