title: hibernate命名策略 

#  Hibernate命名策略 
Java驼峰命名转db下划线命名：
如java实体类 UserName到db表名user_name
```

	<!-- Jpa Entity Manager 配置 -->
	<bean id="entityManagerFactory"
		class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="jpaVendorAdapter">
			<bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
				<property name="showSql" value="false" />
				<property name="generateDdl" value="false" />
				<property name="database" value="MYSQL" />
			</bean>
		</property>
		<!--Weblogic/Jboss这些自带JPA支持的应用服务器有时候会扫描persistence.xml，因此彻底删除掉这个文件需要加，否则会报错 -->
		<property name="packagesToScan" value="com.css.**.entity" />
		<property name="jpaProperties">
			<props>
				<!-- 命名规则 My_NAME->MyName java驼峰变db下划线命名-->
				<prop key="hibernate.ejb.naming_strategy">org.hibernate.cfg.ImprovedNamingStrategy</prop>
			</props>
		</property>
	</bean>

```