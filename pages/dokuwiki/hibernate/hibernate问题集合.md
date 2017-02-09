title: hibernate问题集合 

#  Hibernate问题集合 
##  Hibernate无法自动建表问题 
方言改用` org.hibernate.dialect.MySQL5Dialect `而非org.hibernate.dialect.MySQLInnoDBDialect
<prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>