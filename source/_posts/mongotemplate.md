---
title: mongotemplate
author: ziyi
tags:
  - spring boot
  - mongotemplate
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - mongotemplate
comments: true
---

#### mongotemplate
##### pom.xml

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

##### 配置
`spring.data.mongodb.uri=mongodb://xxxx:27017/xxxx`

##### 使用MongoTemplate查询
- 单条件
	
`Query query = Query.query(Criteria.where("id").is("1"));`
	
- or & and

```
Criteria criteria = new Criteria();

criteria.orOperator(
Criteria.where("title").is("a"),
Criteria.where("name").is("b"));

criteria.andOperator(
Criteria.where("time").ge(start),
Criteria.where("time").is(end));
Query query = new Query(criteria);
```
##### 一个字段多约束需要andOperator

- sort

```
Query query = new Query();
Sort sort = new Sort(new Sort.Order(Sort.Direction.DESC,"id"));
query.with(sort);
```
##### 2.x后是Sort.by

- 分页

```
Query query = new Query();
query.skip(1).limit(3);
```

- update

```
Update update = new Update();
update.set("name","a");
update.set("title","b");
UpdateResult wr = mongoTemplate.updateFirst(query, update, collect.class);
```

##### ne为不等于
##### not为字段不存在

- 正则匹配查询

```
query.addCriteria(Criteria.where("key").regex(".*?\\" + value + ".*")
```

- 分次批量拉取数据 不用分页 因为分页到后期会很慢

```
/**
     * 获取id后的某一时间段内的size个的任务绩效
     * @param id  （mongo中存储的id）
     * @param startTime
     * @param endTime
     * @return
     */
    public <T extends PerformanceEntity> List<T> getTaskPerformanceList(String id, Date startTime, Date endTime, Class<T> t) {
        return getTaskPerformanceList(size, id, startTime, endTime, t);
    }

    /**
     * 获取id后的某一时间段内的size个的标注/质检任务绩效
     * @param size
     * @return
     */
    public <T extends PerformanceEntity> List<T> getTaskPerformanceList(int size, String id, Date startTime, Date endTime, Class<T> t) {
        Query query = new Query();
        Criteria criteria = new Criteria();
        criteria.andOperator(Criteria.where(CREATE_TIME).gte(startTime), Criteria.where(CREATE_TIME).lt(endTime));
        query.addCriteria(criteria);
        ObjectId objectId;
        if (StringUtils.isNotBlank(id)) {
            objectId = new ObjectId(id);
            query.addCriteria(Criteria.where(ID).gt(objectId));
        }
        query.with(Sort.by(Sort.Direction.ASC, ID));
        query.limit(size);
        return mongoTemplate.find(query, t);
    }
```
##### id第一次传null