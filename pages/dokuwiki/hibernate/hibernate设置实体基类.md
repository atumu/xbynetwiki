title: hibernate设置实体基类 

#  Hibernate设置实体基类 
```

/**
 * entity实体类的基类，提供一些固定的字段,继承此类的实体类都是逻辑删除
 */
@MappedSuperclass
public abstract class BaseEntity implements Serializable{
    private static final long serialVersionUID = -6669589572362024270L;

    /**
     * entity基类不约束表的物理主键，实体类自行填写物理主键，示例如下：<br>
     * 
     * @Id
     * @GeneratedValue(generator = "paymentableGenerator")
     * @GenericGenerator(name = "paymentableGenerator", strategy = "uuid")
     * private String uuid;
     */

    /**
     * 所有表一律增加一个创建时间字段
     */
    @Column(name = "CREATE_TIME")
    private Timestamp createTime;

    /**
     * 所有表一律增加一个最后修改时间字段
     */
    @Column(name = "LASTMODIFY_TIME")
    @JSONField(serialize = false)
    private Timestamp lastModifyTime;

    /**
     * @return the lastModifyTime
     */
    public Timestamp getLastModifyTime() {
        return lastModifyTime;
    }

    /**
     * @param lastModifyTime
     *            the lastModifyTime to set
     */
    public void setLastModifyTime(Timestamp lastModifyTime) {
        this.lastModifyTime = lastModifyTime;
    }

    public void setLastModifyTime(long lastModifyTime) {
        this.lastModifyTime = new Timestamp(lastModifyTime);
    }

    /**
     * @return the lastModifyUser
     */
    public String getLastModifyUser() {
        return lastModifyUser;
    }

    /**
     * @return the createTime
     */
    public Timestamp getCreateTime() {
        return createTime;
    }

    /**
     * @param createTime
     *            the createTime to set
     */
    public void setCreateTime(Timestamp createTime) {
        this.createTime = createTime;
    }

    public void setCreateTime(long createTime) {
        this.createTime = new Timestamp(createTime);
    }


    /**
     * @return the serialversionuid
     */
    public static long getSerialversionuid() {
        return serialVersionUID;
    }

}

```