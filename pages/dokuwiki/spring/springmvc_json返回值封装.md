title: springmvc_json返回值封装 

#  SpringMVC JSON返回值封装 
仅仅是一个示列：为了便于所有返回json的地方进行统一：方便前台获取数据
```

import java.io.Serializable;
import java.util.List;

import com.alibaba.fastjson.annotation.JSONField;

/**
 * @description 统一返回数据
 * @createTime 2014年9月21日 下午5:19:57
 * @author tungSing
 * @version 1.0
 */
public class ResultData<T> implements Serializable {

    /**
     * 逻辑正确
     */
    public static final String RESULT_STATUS_OK = "200";

    /**
     * 权限不足，禁止访问
     */
    public static final String RESULT_STATUS_FORBIDDEN = "403";

    /**
     * 服务器发生错误
     */
    public static final String RESULT_STATUS_ERROR = "500";

    /**
     * session out
     */
    public static final String RESULT_SESSION_TIMEOUT = "800";

    private static final long serialVersionUID = 571887313798592339L;

    // grid列表对象
    @JSONField(name = "curPageData")
    // 前台接收时的名称
    private List<T> rows;

    // 总数
    @JSONField(name = "allDataCount")
    // 前台接收时的名称
    private Long total;

    /**
     * 状态200 404 403
     */
    private String status;

    // 实体对象
    private T entity;

    private ResultData(Builder<T> builder) {
        this.rows = builder.rows;
        this.total = builder.total;
        this.status = builder.status;
        this.entity = builder.entity;
        this.msg=builder.msg;
    }

    public static class Builder<T> {
        private List<T> rows;

        private Long total;

        private String status = ResultData.RESULT_STATUS_OK;

        private T entity;
        private String msg="";

        public Builder() {
        }

        public Builder<T> setRows(List<T> rows) {
            this.rows = rows;
            return this;
        }

        public Builder<T> setTotal(Long total) {
            this.total = total;
            return this;
        }

        public Builder<T> setStatus(String status) {
            this.status = status;
            return this;
        }

        public Builder<T> setEntity(T entity) {
            this.entity = entity;
            return this;
        }

        // 构造器入口
        public ResultData<T> build() {
            return new ResultData<T>(this);
        }

    }

    public List<T> getRows() {
        return rows;
    }

    public Long getTotal() {
        return total;
    }

    public String getStatus() {
        return status;
    }

    public T getEntity() {
        return entity;
    }

	

}

```
