title: sqlalchemy db.create_all 无法创建部分model
date: 2016-09-01 20:34:57
tags: [Flask,SQLALChemy,Python]
categories: Flask
---
https://github.com/mattshma/omega/issues/8

String 没有加字段限制

from web.report.models import *
可以创建表　但是映射关系没了
InvalidRequestError: When initializing mapper Mapper|Auction|auction, expression
    'Item' failed to locate a name ("name 'Item' is not defined"). If this is a
    class name, consider adding this relationship() to the Auction class after
    both dependent classes have been defined.
