title: hibernate分页查询 

#  Hibernate分页查询 
主要涉及以下两个方法
```

query.setFirstResult((pagenoum-1)*page_no);
query.setMaxResults(page_no);

```
部分代码如下：
```

/**
*page_size:每页的数据量
*currentPageNoum:当前需要查询的页面号
*数据库row是从1开始，页面号也是从1开始。
*/
public class hibernateOrderDaoImpl  {
    private static final int page_size = 2;
    public List findAllOrders(int currentPageNoum) {
        Session session = null;
        List orderList = null;
        Query query = null;
        try{
            session = HibernateUtil.getSession();
            session.getTransaction().begin();
            query = session.createQuery("select o from Order o order by o.id");
             //设置开始查询的行，跳过以前已经查询过的那些行
            query.setFirstResult((currentPageNoum-1)*page_size);
            //设置此次查询的最大行数
            query.setMaxResults(page_size);
            orderList = query.list();
            session.getTransaction().commit(); 
        }catch(Exception e){   
            e.printStackTrace();
            session.getTransaction().rollback();
        }finally{  
            session.close();
        }
        return orderList;
    }
    public static void main(String[] args) {
      //每页2行数据，分3页。
        hibernateOrderDaoImpl dao = new hibernateOrderDaoImpl();
        List list = dao.findAllOrders(3);
        Iterator it = list.iterator();
        while(it.hasNext()){   
            Order order = (Order) it.next();
            System.out.println(order.getId());
        }
    }  
}

```

这里有个分页的示例代码：http://www.cnblogs.com/xiaoluo501395377/archive/2012/10/18/2730073.html
参考：
http://blog.163.com/magicc_love/blog/static/185853662201282141433406/
http://gaogengzhi.iteye.com/blog/266301