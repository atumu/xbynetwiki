title: spring_data_jpa自定义仓库 

#  spring data jpa自定义仓库 
配置:
```

 <!-- 配置Spring Data JPA扫描目录-->  
    <jpa:repositories base-package="com.xby.**.dao"  
        entity-manager-factory-ref="entityManagerFactory"  
        transaction-manager-ref="transactionManager"  
        repository-impl-postfix="Impl"  
        factory-class="com.xby.dao.DefaultRepositoryFactoryBean">  
    </jpa:repositories>  

```
1、扩展JpaRepository接口，将来所有dao将继承该接口以实现DAO的封装
```

/**
 * 针对spring data jpa所提供的接口{@link JpaRepository}再次扩展
 * @NoRepositoryBean是必须的.防止被Spring data JPA当作一个仓库而被提供了实现，它仅仅只是一个仓库接口。
 */
@NoRepositoryBean
public interface GenericRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
	//JpaRepository本身是一个空接口，下面所有的方法声明都是自定义的
	 List<T> findAll(HashMap<String,Object> queryParams,  
            LinkedHashMap<String, String> orderby);  
    SimplePage<T> findByPage(HashMap<String,Object> queryParams,  
            LinkedHashMap<String, String> orderby, Integer pageSize,Integer pageNum);  
      
    SimplePage<T> findByPageWithSql(String sql,  
            HashMap<String, Object> queryParams, Integer pageSize,  
            Integer pageNum);  
  
    SimplePage<T> findByPageWithWhereHql(String whereHql,  
            HashMap<String, Object> queryParams, Integer pageSize,  
            Integer pageNum);  
  
  
    SimplePage<T> findByPage(HashMap<String, Object> queryParams,  
            String orderby, Integer pageSize, Integer pageNum);  
  
    /** 
     * 根据sql语句查询，结果是实体的集合  
     * @param sql 
     * @param entityClass 
     * @return 
     */  
  
    public List<T> findAllBySql(Class<T> entityClass, String sql);  
      
    public String getUniqueResultBySql(String sql,HashMap<String,Object> queryParams);  
      
    //public List<T> findByAttachIds(String[] ids);  
}

```
2、自定义扩展实现类：GenericRepositoryImpl ，其中的其他方法都是全局共享方法，即每个继承自定义扩展接口的用户接口均可使用这些函数
```

@NoRepositoryBean   // 必须的  
//必须继承 SimpleJpaRepository这个JpaRepository的默认实现
public class GenericRepositoryImpl<T, ID extends Serializable> extends  
        SimpleJpaRepository<T, ID> implements GenericRepository<T, ID> {  
  
    static Logger logger = Logger.getLogger(GenericRepositoryImpl.class);  
    private final EntityManager em;  
    private final Class<T> entityClass;  
    private final String entityName;  
    private final JpaEntityInformation<T, ?> entityInformation; //非常有用，如判断某个实体是否为新加的
    private CrudMethodMetadata crudMethodMetadata;
      
  
    /** 
     * 构造函数 
     * @param domainClass 
     * @param em 
     */  
    public GenericRepositoryImpl(final JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {  
          
        super(entityInformation, entityManager);  
        this.em = entityManager;  
        this.entityClass=entityInformation.getJavaType();  
        this.entityName=entityInformation.getEntityName();  
    }  
      
    /** 
     * 构造函数 
     * @param domainClass 
     * @param em 
     */  
    public GenericRepositoryImpl(Class<T> domainClass, EntityManager em) {  
        this(JpaEntityInformationSupport.getMetadata(domainClass, em), em);   
    } 
     public GenericRepositoryImpl(final JpaEntityInformation<T, ?> ei, EntityManager em) {

        super(ei, em);
        entityManager = em;
        entityInformation = ei;
    }      
      
    @Override  
    public List<T> findAll(HashMap<String,Object> queryParams,  
            LinkedHashMap<String, String> orderby){          
           
        String whereHql=buildWhereQuery(queryParams);  
        String orderHql=buildOrderby(orderby);  
      
        String hql = "select o from "+entityName+" o where o.removed=0";   
        Query query=createQuery(hql+whereHql+orderHql,queryParams);  
  
        List<T> list=(List<T>)query.getResultList();  
        return list;      
    }  
      
      
    @Override  
    public SimplePage<T> findByPage(HashMap<String,Object> queryParams,  
            LinkedHashMap<String, String> orderby, Integer pageSize,Integer pageNum) {  
          
        return findByPage(queryParams,buildOrderby(orderby), pageSize, pageNum);  
  
    }  
    @Override  
    public SimplePage<T> findByPage(HashMap<String,Object> queryParams,  
            String orderby, Integer pageSize,Integer pageNum) {  
                       
        String whereHql=buildWhereQuery(queryParams);  
        String orderHql=orderby;  
          
        String hql="select count(*) from "+ entityName+ " o where o.removed=0 ";   
        Query query=createQuery(hql+whereHql+orderHql,queryParams);  
        PageInfo pageInfo=new PageInfo(((Long)query.getSingleResult()).intValue(),pageSize);  
        pageInfo.refresh(pageNum);  
          
  
        hql = "select o from "+ entityName+ " o where o.removed=0 ";  
        query=createQuery(hql+whereHql+orderHql,queryParams);  
        query.setFirstResult(pageInfo.getStartRecord()).setMaxResults(pageInfo.getPageSize());    
  
        return new SimplePage<T>(pageInfo,query.getResultList());  
    }  
      
    private Query createQuery(String hql,HashMap<String,Object> queryParams){  
        Query query = em.createQuery(hql);  
        setQueryParams(query, queryParams);  
        return query;  
    }  
    @Override  
    public SimplePage<T> findByPageWithWhereHql(String whereHql, HashMap<String,Object> queryParams, Integer pageSize,Integer pageNum) {  
          
        String hql="select count(*) from "+ entityName+ " o where o.removed=0 ";    
        if(whereHql==null){  
            whereHql="";  
        }  
  
        Query query = em.createQuery(hql+whereHql);  
        setQueryParams(query, queryParams);  
        PageInfo pageInfo=new PageInfo(((Long)query.getSingleResult()).intValue(),pageSize);  
        pageInfo.refresh(pageNum);  
          
  
        hql = "select o from "+ entityName+ " o where o.removed=0 ";  
        query = em.createQuery(hql+whereHql);          
        setQueryParams(query, queryParams);  
        query.setFirstResult(pageInfo.getStartRecord()).setMaxResults(pageInfo.getPageSize());    
  
        return new SimplePage<T>(pageInfo,query.getResultList());  
    }  
      
    @Override  
    public SimplePage<T> findByPageWithSql(String sql, HashMap<String,Object> queryParams, Integer pageSize,Integer pageNum) {  
  
        Query query = em.createNativeQuery("select count(*) from ("+sql+")");    
        setQueryParams(query, queryParams);  
          
        PageInfo pageInfo=new PageInfo(((Long)query.getSingleResult()).intValue(),pageSize);  
        pageInfo.refresh(pageNum);  
          
        query = em.createNativeQuery(sql);  
        setQueryParams(query, queryParams);  
        query.setFirstResult(pageInfo.getStartRecord()).setMaxResults(pageInfo.getPageSize());    
  
        return new SimplePage<T>(pageInfo,query.getResultList());  
    }  
      
    private void setQueryParams(Query query, HashMap<String, Object> queryParams){  
        if(queryParams!=null && queryParams.size()>0){  
            for(String key : queryParams.keySet()){  
                Class clazz=ReflectUtil.getFieldType(this.entityClass, key.replaceAll("_s","").replaceAll("_e", "").replaceAll("_in", ""));  
                if(clazz!=null && clazz.equals(String.class) && !key.endsWith("_in")){  
                    query.setParameter(key, '%'+queryParams.get(key).toString()+'%');  
                }else{  
                    query.setParameter(key, queryParams.get(key));  
                }  
  
            }  
        }  
    }  
  
    private String buildOrderby(LinkedHashMap<String, String> orderby) {  
        // TODO Auto-generated method stub  
        StringBuffer orderbyql = new StringBuffer("");  
        if(orderby!=null && orderby.size()>0){  
            orderbyql.append(" order by ");  
            for(String key : orderby.keySet()){  
                orderbyql.append("o.").append(key).append(" ").append(orderby.get(key)).append(",");  
            }  
            orderbyql.deleteCharAt(orderbyql.length()-1);  
        }  
          
        return orderbyql.toString();  
    }  
      
    private String buildWhereQuery(HashMap<String, Object> queryParams) {  
        // TODO Auto-generated method stub  
        StringBuffer whereQueryHql = new StringBuffer("");  
        if(queryParams!=null && queryParams.size()>0){  
            for(String key : queryParams.keySet()){  
//              if(key.equalsIgnoreCase("id")){  
//                  whereQueryHql.append(" and ").append("o.").append(key).append(" in(:").append(key).append(")");  
//              }else   
                if(key.endsWith("_s")){  
                    whereQueryHql.append(" and ").append("o.").append(key.replace("_s", "")).append(" >=:").append(key);  
                }else if(key.endsWith("_e")){  
                    whereQueryHql.append(" and ").append("o.").append(key.replace("_e", "")).append(" <=:").append(key);  
                }else if(key.endsWith("_in")){  
                    whereQueryHql.append(" and ").append("o.").append(key.replace("_in", "")).append(" in:").append(key);  
                }else{  
                    Class clazz=ReflectUtil.getFieldType(this.entityClass, key);  
                    if(clazz!=null && clazz.equals(String.class)){  
                        whereQueryHql.append(" and ").append("o.").append(key).append(" like :").append(key);  
                    }else{  
                        whereQueryHql.append(" and ").append("o.").append(key).append(" =:").append(key);  
                    }  
                      
                }  
            }  
        }  
          
        return whereQueryHql.toString();  
    }  
  
    @Override  
    public List<T> findAllBySql(Class<T> entityClass, String sql) {  
          //创建原生SQL查询QUERY实例,指定了返回的实体类型  
          Query query =  em.createNativeQuery(sql,entityClass);  
          //执行查询，返回的是实体列表,  
          List<T> EntityList = query.getResultList();  
        return EntityList;  
    }  
  
    public String getUniqueResultBySql(String sql,HashMap<String,Object> queryParams){  
        Query query =  em.createNativeQuery(sql);  
        for(String key : queryParams.keySet()){  
            query.setParameter(key, queryParams.get(key));  
        }  
        //执行查询，返回的是实体列表,  
        String result = (String)query.getSingleResult();  
       return result;  
    }  
}

```
3、工厂bean：DefaultRepositoryFactoryBean
```

public class DefaultRepositoryFactoryBean<T extends JpaRepository<S, ID>, S, ID extends Serializable>  
        extends JpaRepositoryFactoryBean<T, S, ID> {  
    protected RepositoryFactorySupport createRepositoryFactory(EntityManager entityManager) {  
        return new DefaultRepositoryFactory(entityManager);  
    }  
}  

```
4、工厂类：DefaultRepositoryFactory
```

public class DefaultRepositoryFactory extends JpaRepositoryFactory {  
  
    private final EntityManager entityManager;  
      
    public DefaultRepositoryFactory(EntityManager entityManager) {  
        super(entityManager);  
        Assert.notNull(entityManager);  
        this.entityManager = entityManager;  
          
    }  
      
   @Override  
    protected <T, ID extends Serializable> SimpleJpaRepository<?, ?> getTargetRepository(RepositoryMetadata metadata, EntityManager entityManager) {  
        //TODO  
        JpaEntityInformation<?, Serializable> entityInformation = getEntityInformation(metadata.getDomainType());  
        return new GenericRepositoryImpl(entityInformation, entityManager); // custom implementation  
    }  
    
    @Override  
    protected Class<?> getRepositoryBaseClass(RepositoryMetadata metadata) {  
        return GenericRepositoryImpl.class;  
    }  
}  

```
5、使用范例：自定义接口继承 扩展接口
```

public interface CpqMainDao extends GenericRepository<CpqMain, Integer>{  
  
    public Page<CpqMain> findAll(Pageable pageable);  
          
    @Query("select a from CpqMain a where a.removed = 0 and a.id =?")  
    public CpqMain findById(Integer id);  
      
    @Transactional    
    @Modifying   
    @Query("update CpqMain a set a.removed = 1 where a.id =:id")  
    public Integer removeById(@Param("id")Integer id);  
      
      
    @QueryHints({ @QueryHint(name = "org.hibernate.cacheable", value ="true") })    
    public List<CpqMain> findAll();    
  
      
}  

```


##  给实体添加创建和修改时间 
```

@NoRepositoryBean
public class SDCRepositoryImpl<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements SDCRepository<T, ID> {

    private final EntityManager entityManager;

    private final JpaEntityInformation<T, ?> entityInformation;

    private CrudMethodMetadata crudMethodMetadata;

    /**
     * 构造函数
     * 
     * @param domainClass
     * @param em
     */
    public SDCRepositoryImpl(final JpaEntityInformation<T, ?> ei, EntityManager em) {

        super(ei, em);
        entityManager = em;
        entityInformation = ei;
    }

    @Override
    @Transactional
    public <S extends T> S save(S entity) {
        if (entity instanceof CommonEntity) {
            CommonEntity baseEntity = (CommonEntity) entity;
            if (!entityInformation.isNew(entity)) {  //检测实体是否为新加的
                baseEntity.setOperStatus(CommonEntity.OPER_STATUS_UPDATE);
            } else {
                baseEntity.setCreateTime(System.currentTimeMillis()); .//如果是新加的那么设置创建时间和用户
                baseEntity.setCreateUser(UserThreadLocal.get().getUserId());
            }
            baseEntity.setInvalid(IF.FALSE_CHAR);
            baseEntity.setLastModifyTime(System.currentTimeMillis()); //设置修改时间
            baseEntity.setLastModifyUser(UserThreadLocal.get().getUserId()); 
            S newEntity = (S) baseEntity;
            return super.save(newEntity);
        } else {
            return super.save(entity);
        }

    }

    @Override
    @Transactional
    public void delete(T entity) {
        if (entity instanceof CommonEntity) {
            CommonEntity baseEntity = (CommonEntity) entity;
            baseEntity.setOperStatus(CommonEntity.OPER_STATUS_DELETE);
            baseEntity.setInvalid(IF.TRUE_CHAR);
            baseEntity.setLastModifyTime(System.currentTimeMillis());
            baseEntity.setLastModifyUser(UserThreadLocal.get().getUserId());

            super.save((T) baseEntity);
        } else {
            super.delete(entity);
        }
    }

    protected TypedQuery<T> getQuery(Specification<T> spec, Sort sort) {

        CriteriaBuilder builder = entityManager.getCriteriaBuilder();
        CriteriaQuery<T> query = builder.createQuery(getDomainClass());

        Root<T> root = applySpecificationToCriteria(spec, query);
        query.select(root);

        if (sort != null) {
            query.orderBy(toOrders(sort, root, builder));
        }

        return applyRepositoryMethodMetadata(entityManager.createQuery(query));
    }

    /**
     * Creates a new count query for the given {@link Specification}.
     * 
     * @param spec
     *            can be {@literal null}.
     * @return
     */
    protected TypedQuery<Long> getCountQuery(Specification<T> spec) {

        CriteriaBuilder builder = entityManager.getCriteriaBuilder();
        CriteriaQuery<Long> query = builder.createQuery(Long.class);

        Root<T> root = applySpecificationToCriteria(spec, query);

        if (query.isDistinct()) {
            query.select(builder.countDistinct(root));
        } else {
            query.select(builder.count(root));
        }

        return entityManager.createQuery(query);
    }

    private <S> Root<T> applySpecificationToCriteria(Specification<T> spec, CriteriaQuery<S> query) {

        Assert.notNull(query);
        Class<T> clazz = getDomainClass();
        Root<T> root = query.from(clazz);
        CriteriaBuilder builder = entityManager.getCriteriaBuilder();
        Predicate invalidPredicate = null;
        if (CommonEntity.class.isAssignableFrom(clazz)) {
            // 是类属性名，不是字段名称
            Path invalidPath = root.get("invalid");
            invalidPredicate = builder.equal(invalidPath, IF.FALSE_CHAR);
        }

        if (spec == null) {
            if (invalidPredicate != null) {
                query.where(invalidPredicate);
            }

            return root;
        }

        Predicate predicate = spec.toPredicate(root, query, builder);

        if (predicate != null) {
            if (invalidPredicate != null) {
                predicate = builder.and(predicate, invalidPredicate);
            }
            query.where(predicate);
        }

        return root;
    }

    private TypedQuery<T> applyRepositoryMethodMetadata(TypedQuery<T> query) {

        if (crudMethodMetadata == null) {
            return query;
        }

        LockModeType type = crudMethodMetadata.getLockModeType();
        TypedQuery<T> toReturn = type == null ? query : query.setLockMode(type);

        for (Entry<String, Object> hint : crudMethodMetadata.getQueryHints().entrySet()) {
            query.setHint(hint.getKey(), hint.getValue());
        }

        return toReturn;
    }

}


```
参考：http://blog.csdn.net/z69183787/article/details/42528397