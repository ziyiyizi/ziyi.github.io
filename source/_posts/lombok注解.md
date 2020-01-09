---
title: lombok注解
author: ziyi
date: 2019-12-26 23:32:30
tags:
  - spring boot
  - lombok
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - spring lombok
comments: true
---

#### 常用的lombok注解
- @Data

> 包含了@ToString @EqualsAndHashCode @Getter @Setter
> @ToString等可使用callSuper=true来引入父类属性 用exclude来排除属性

- @Value
> 跟@Data差不多 但所有属性会为final

- @NoArgsConstructor
> 无参构造器 可用access = AccessLevel.PRIVATE来标识这是个私有构造器

- @AllArgsConstructor

- @Accessors(chain = true)
> 对象链式构造

- @Builder

- @Slf4J

##### idea使用lombok
plugins 安装lombok插件

##### pom引入依赖
```
<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.18.10</version>
		<scope>provided</scope>
	  </dependency>
```