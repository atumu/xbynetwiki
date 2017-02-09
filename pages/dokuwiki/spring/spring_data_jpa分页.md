title: spring_data_jpa分页 

#  spring data jpa分页 
利用Spring提供的PageRequest这个类实现了Pageable接口.
```

@Component
public interface TaskDao extends PagingAndSortingRepository<Task, Long>, JpaSpecificationExecutor<Task> {
 
    Page<Task> findByUserId(Long id, Pageable pageRequest);
 
    @Modifying
    @Query("delete from Task task where task.user.id=?1")
    void deleteByUserId(Long id);
}

```
```

public Page<Task> getUserTask(Long userId, Map<String, Object> searchParams, int pageNumber, int pageSize,
            String sortType) {
        PageRequest pageRequest = buildPageRequest(pageNumber, pageSize, sortType);
        Specification<Task> spec = buildSpecification(userId, searchParams);
 
        return taskDao.findAll(spec, pageRequest);
    }
 
    /**
     * 创建分页请求.
     */
    private PageRequest buildPageRequest(int pageNumber, int pagzSize, String sortType) {
        Sort sort = null;
        if ("auto".equals(sortType)) {
            sort = new Sort(Direction.DESC, "id");
        } else if ("title".equals(sortType)) {
            sort = new Sort(Direction.ASC, "title");
        }
 
        return new PageRequest(pageNumber - 1, pagzSize, sort);
    }
 
    /**
     * 创建动态查询条件组合.
     */
    private Specification<Task> buildSpecification(Long userId, Map<String, Object> searchParams) {
        Map<String, SearchFilter> filters = SearchFilter.parse(searchParams);
        filters.put("user.id", new SearchFilter("user.id", Operator.EQ, userId));
        Specification<Task> spec = DynamicSpecifications.bySearchFilter(filters.values(), Task.class);
        return spec;
    }

```