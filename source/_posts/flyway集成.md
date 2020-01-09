---
title: flyway集成
author: ziyi
tags:
  - spring boot
  - flyway
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - flyway
comments: true
---

# flyway集成

- 引入pom依赖

```
<dependency>
            <groupId>org.flywaydb</groupId>
            <artifactId>flyway-core</artifactId>
            <version>6.1.0</version>
        </dependency>
```

- properties 可有

```
# 设为false，不对已经执行的sql进行内容校验
spring.flyway.validate-on-migrate=false
```

- resource文件夹下新加db文件夹 在db文件夹下新建migration文件夹 在migration文件夹下新建sql初始脚本 命名格式"VyyMMddHHmm__init.sql" 如下

```
USE `label_datastore`;

SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;


CREATE TABLE IF NOT EXISTS `t_user`
(
    `id`               VARCHAR(32) NOT NULL COMMENT '用户ID',
    `userName`         VARCHAR(255) DEFAULT NULL COMMENT '用户名',
    PRIMARY KEY (`id`) USING BTREE
)
    ENGINE = InnoDB
    CHARACTER SET = utf8mb4
    COLLATE = utf8mb4_unicode_ci
    ROW_FORMAT = Dynamic
    COMMENT = '【用户】表';
```