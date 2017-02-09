title: hibernate自动更新时间问题 

#  Hibernate自动更新时间问题 
实现方法一:
updatetime字段表结构设计    
`updatetime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
hibernate类中对应的实体类中不加该字段

实现方法二：
updatetime字段表结构设计   
`updatetime` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

当你实体类中需要用到updatetime，加updatetime字段时
必须添加如下注解，表示该字段更新不受实体类的影响，由数据库自动更新
```

    @Temporal(TemporalType.TIMESTAMP)
    @Column(insertable = false, updatable = false)  
    @org.hibernate.annotations.Generated(org.hibernate.annotations.GenerationTime.ALWAYS) 
    private Date updatetime;

```

方法三：
```

@PreUpdate public void preUpdate() { modifyTime = new Date(); }

```