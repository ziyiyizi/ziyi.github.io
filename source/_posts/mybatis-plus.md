---
title: mybatis plus
author: ziyi
tags:
  - spring boot
  - mybatis plus
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - mybatis plus
comments: true
---

#### Mybatis Plus
##### 引入依赖
```
<dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.2.0</version>
</dependency>
```

##### yml配置文件

```
spring.datasource.url=jdbc:mysql://xxxx:3306/xxxx?allowMultiQueries=true
spring.datasource.username=root
spring.datasource.password=xxxx
```

##### 启动类加上MapperScan注解
`@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")`

##### 编写实体类
```
@Data
public abstract class BaseSqlEntity {

    /**
     * 主键
     */
    @TableId(type = IdType.ID_WORKER)
    private Long id;

    /**
     * 创建时间
     */
    private Date createTime;

    /**
     * 创建者
     */
    private String createUser;


    /**
     * insert实体时调用
     */
    public void setCreateFields(String userId) {
        this.createTime = new Date();
        this.createUser = userId;
    }

}
@TableName("t_user")
@Data
public class User extends BaseSqlEntity {

    /**
     * 账户名
     */
    private String userName;
}
```

##### 编写Mapper
```
public interface UserMapper extends BaseMapper<User> {

}
```

##### 测试类
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        Assert.assertEquals(5, userList.size());
        userList.forEach(System.out::println);
    }

}
```

##### 分页插件 启动类配置
```
@Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        return paginationInterceptor;
    }
```

##### 官方教程
https://mp.baomidou.com/

##### note
- 使用lambdaQueryMapper lambdaUpdateMapper
- selectOne需要手动加上last('limit 1')
- 分页从1开始
