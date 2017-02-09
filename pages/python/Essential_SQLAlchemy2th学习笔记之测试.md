modifyAt:2016-12-17 16:38:34
title:Essential SQLAlchemy2th学习笔记之测试
createAt:2016-12-17 14:32:16
location:python/Essential_SQLAlchemy2th学习笔记之测试
author:xbynet

# Core模块测试
## Testing with a Test Database
数据访问层:

```
from datetime import datetime
from sqlalchemy import (MetaData, Table, Column, Integer, Numeric, String,
        DateTime, ForeignKey, Boolean, create_engine)
from sqlalchemy.sql import insert


class DataAccessLayer:
    connection = None
    engine = None
    conn_string = None
    metadata = MetaData()
    cookies = Table('cookies',
        metadata,
        Column('cookie_id', Integer(), primary_key=True),
        Column('cookie_name', String(50), index=True),
        Column('cookie_recipe_url', String(255)),
        Column('cookie_sku', String(55)),
        Column('quantity', Integer()),
        Column('unit_cost', Numeric(12, 2))
    )

    users = Table('users', metadata,
        Column('user_id', Integer(), primary_key=True),
        Column('customer_number', Integer(), autoincrement=True),
        Column('username', String(15), nullable=False, unique=True),
        Column('email_address', String(255), nullable=False),
        Column('phone', String(20), nullable=False),
        Column('password', String(25), nullable=False),
        Column('created_on', DateTime(), default=datetime.now),
        Column('updated_on', DateTime(), default=datetime.now, onupdate=datetime.now)
    )

    orders = Table('orders', metadata,
        Column('order_id', Integer()),
        Column('user_id', ForeignKey('users.user_id')),
        Column('shipped', Boolean(), default=False)
    )

    line_items = Table('line_items', metadata,
        Column('line_items_id', Integer(), primary_key=True),
        Column('order_id', ForeignKey('orders.order_id')),
        Column('cookie_id', ForeignKey('cookies.cookie_id')),
        Column('quantity', Integer()),
        Column('extended_cost', Numeric(12, 2))
    )

    def db_init(self, conn_string):
        self.engine = create_engine(conn_string or self.conn_string)
        self.metadata.create_all(self.engine)
        self.connection = self.engine.connect()

dal = DataAccessLayer()


def prep_db():
    ins = dal.cookies.insert()
    dal.connection.execute(ins, cookie_name='dark chocolate chip',
            cookie_recipe_url='http://some.aweso.me/cookie/recipe_dark.html',
            cookie_sku='CC02',
            quantity='1',
            unit_cost='0.75')
    inventory_list = [
        {
            'cookie_name': 'peanut butter',
            'cookie_recipe_url': 'http://some.aweso.me/cookie/peanut.html',
            'cookie_sku': 'PB01',
            'quantity': '24',
            'unit_cost': '0.25'
        },
        {
            'cookie_name': 'oatmeal raisin',
            'cookie_recipe_url': 'http://some.okay.me/cookie/raisin.html',
            'cookie_sku': 'EWW01',
            'quantity': '100',
            'unit_cost': '1.00'
        }
    ]
    dal.connection.execute(ins, inventory_list)

    customer_list = [
        {
            'username': "cookiemon",
            'email_address': "mon@cookie.com",
            'phone': "111-111-1111",
            'password': "password"
        },
        {
            'username': "cakeeater",
            'email_address': "cakeeater@cake.com",
            'phone': "222-222-2222",
            'password': "password"
        },
        {
            'username': "pieguy",
            'email_address': "guy@pie.com",
            'phone': "333-333-3333",
            'password': "password"
        }
    ]
    ins = dal.users.insert()
    dal.connection.execute(ins, customer_list)
    ins = insert(dal.orders).values(user_id=1, order_id='wlk001')
    dal.connection.execute(ins)
    ins = insert(dal.line_items)
    order_items = [
        {
            'order_id': 'wlk001',
            'cookie_id': 1,
            'quantity': 2,
            'extended_cost': 1.00
        },
        {
            'order_id': 'wlk001',
            'cookie_id': 3,
            'quantity': 12,
            'extended_cost': 3.00
        }
    ]
    dal.connection.execute(ins, order_items)
    ins = insert(dal.orders).values(user_id=2, order_id='ol001')
    dal.connection.execute(ins)
    ins = insert(dal.line_items)
    order_items = [
        {
            'order_id': 'ol001',
            'cookie_id': 1,
            'quantity': 24,
            'extended_cost': 12.00
        },
        {
            'order_id': 'ol001',
            'cookie_id': 4,
            'quantity': 6,
            'extended_cost': 6.00
        }
    ]
    dal.connection.execute(ins, order_items)

```

需要测试的逻辑;

```
from db import dal
from sqlalchemy.sql import select


def get_orders_by_customer(cust_name, shipped=None, details=False):
    columns = [dal.orders.c.order_id, dal.users.c.username, dal.users.c.phone]
    joins = dal.users.join(dal.orders)
    if details:
        columns.extend([dal.cookies.c.cookie_name,
                        dal.line_items.c.quantity,
                        dal.line_items.c.extended_cost])
        joins = joins.join(dal.line_items).join(dal.cookies)
    cust_orders = select(columns)
    cust_orders = cust_orders.select_from(joins).where(
        dal.users.c.username == cust_name)
    if shipped is not None:
        cust_orders = cust_orders.where(dal.orders.c.shipped == shipped)
    result = dal.connection.execute(cust_orders).fetchall()
    return result

```

## 单元测试：

```
import unittest

from decimal import Decimal

from db import dal, prep_db
from app import get_orders_by_customer


class TestApp(unittest.TestCase):
    cookie_orders = [(u'wlk001', u'cookiemon', u'111-111-1111')]
    cookie_details = [
        (u'wlk001', u'cookiemon', u'111-111-1111',
            u'dark chocolate chip', 2, Decimal('1.00')),
        (u'wlk001', u'cookiemon', u'111-111-1111',
            u'oatmeal raisin', 12, Decimal('3.00'))]

    @classmethod
    def setUpClass(cls):
        dal.db_init('sqlite:///:memory:')
        prep_db()

    def test_orders_by_customer_blank(self):
        results = get_orders_by_customer('')
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_shipped(self):
        results = get_orders_by_customer('', True)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_notshipped(self):
        results = get_orders_by_customer('', False)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_details(self):
        results = get_orders_by_customer('', details=True)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_shipped_details(self):
        results = get_orders_by_customer('', True, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_notshipped_details(self):
        results = get_orders_by_customer('', False, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust(self):
        results = get_orders_by_customer('bad name')
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_shipped(self):
        results = get_orders_by_customer('bad name', True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_notshipped(self):
        results = get_orders_by_customer('bad name', False)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_details(self):
        results = get_orders_by_customer('bad name', details=True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_shipped_details(self):
        results = get_orders_by_customer('bad name', True, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_bad_cust_notshipped_details(self):
        results = get_orders_by_customer('bad name', False, True)
        self.assertEqual(results, [])

    def test_orders_by_customer(self):
        results = get_orders_by_customer('cookiemon')
        self.assertEqual(results, self.cookie_orders)

    def test_orders_by_customer_shipped_only(self):
        results = get_orders_by_customer('cookiemon', True)
        self.assertEqual(results, [])

    def test_orders_by_customer_unshipped_only(self):
        results = get_orders_by_customer('cookiemon', False)
        self.assertEqual(results, self.cookie_orders)

    def test_orders_by_customer_with_details(self):
        results = get_orders_by_customer('cookiemon', details=True)
        self.assertEqual(results, self.cookie_details)

    def test_orders_by_customer_shipped_only_with_details(self):
        results = get_orders_by_customer('cookiemon', True, True)
        self.assertEqual(results, [])

    def test_orders_by_customer_unshipped_only_details(self):
        results = get_orders_by_customer('cookiemon', False, True)
        self.assertEqual(results, self.cookie_details)

```

运行测试：**# python -m unittest test_app**

## mock测试

```
import unittest
from decimal import Decimal

import mock

from db import dal, prep_db
from app import get_orders_by_customer


class TestApp(unittest.TestCase):
    cookie_orders = [(u'wlk001', u'cookiemon', u'111-111-1111')]
    cookie_details = [
        (u'wlk001', u'cookiemon', u'111-111-1111',
            u'dark chocolate chip', 2, Decimal('1.00')),
        (u'wlk001', u'cookiemon', u'111-111-1111',
            u'oatmeal raisin', 12, Decimal('3.00'))]

    @mock.patch('app.select')
    @mock.patch('app.dal.connection')
    def test_orders_by_customer_blank(self, mock_conn, mock_select):
        mock_select.return_value.select_from.return_value.where.return_value = None
        mock_conn.execute.return_value.fetchall.return_value = []
        results = get_orders_by_customer('')
        self.assertEqual(results, [])

    @mock.patch('app.dal.connection')
    def test_orders_by_customer_blank_shipped(self, mock_conn):
        mock_conn.execute.return_value.fetchall.return_value = []
        results = get_orders_by_customer('', True)
        self.assertEqual(results, [])

    @mock.patch('app.dal.connection')
    def test_orders_by_customer(self, mock_conn):
        mock_conn.execute.return_value.fetchall.return_value = self.cookie_orders
        results = get_orders_by_customer('cookiemon')
        self.assertEqual(results, self.cookie_orders)

```


# ORM模块测试
## 单元测试与Mock
```
import unittest

from decimal import Decimal

from db import prep_db, dal

from app import get_orders_by_customer


class TestApp(unittest.TestCase):
    cookie_orders = [(1, u'cookiemon', u'111-111-1111')]
    cookie_details = [
        (1, u'cookiemon', u'111-111-1111',
            u'dark chocolate chip', 2, Decimal('1.00')),
        (1, u'cookiemon', u'111-111-1111',
            u'oatmeal raisin', 12, Decimal('3.00'))]

    @classmethod
    def setUpClass(cls):
        dal.conn_string = 'sqlite:///:memory:'
        dal.connect()
        dal.session = dal.Session()
        prep_db(dal.session)
        dal.session.close()

    def setUp(self):
        dal.session = dal.Session()

    def tearDown(self):
        dal.session.rollback()
        dal.session.close()

    def test_orders_by_customer_blank(self):
        results = get_orders_by_customer('')
        self.assertEqual(results, [])

    def test_orders_by_customer_blank_shipped(self):
        results = get_orders_by_customer('', True)
        self.assertEqual(results, [])

```

```
import unittest
from decimal import Decimal

import mock

from app import get_orders_by_customer


class TestApp(unittest.TestCase):
    cookie_orders = [(1, u'cookiemon', u'111-111-1111')]
    cookie_details = [
        (1, u'cookiemon', u'111-111-1111',
            u'dark chocolate chip', 2, Decimal('1.00')),
        (1, u'cookiemon', u'111-111-1111',
            u'oatmeal raisin', 12, Decimal('3.00'))]

    @mock.patch('app.dal.session')
    def test_orders_by_customer_blank(self, mock_dal):
        mock_dal.query.return_value.join.return_value.filter.return_value. \
            all.return_value = []
        results = get_orders_by_customer('')
        self.assertEqual(results, [])

```
