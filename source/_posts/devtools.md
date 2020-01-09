---
title: devtools
author: ziyi
tags:
  - spring boot
  - devtools
index_img: /img/mybg.jpg
banner_img: /img/mybg.jpg
categories:
  - spring boot
  - devtools
comments: true
---

### devtools的使用
####原理
> ide在发现代码有更改之后，重新启动应用，但是速度比手动停止后再启动更快。深层原理是使用了两个ClassLoader，一个Classloader加载那些不会改变的类（第三方Jar包），另一个ClassLoader加载会更改的类，称为restart ClassLoader。这样在有代码更改的时候，原来的restart ClassLoader 被丢弃，重新创建一个restart ClassLoader，由于需要加载的类相比较少，所以实现了较快的重启时间。即devtools会监听classpath下的文件变动，并且会立即重启应用（发生在保存时机）

#### 不生效路径
默认情况下，/META-INF/maven，/META-INF/resources，/resources，/static，/templates，/public这些文件夹下的文件修改不会使应用重启，但是会重新加载（devtools内嵌了一个LiveReload server，当资源发生改变时，浏览器刷新

>可以使用spring.devtools.restart.exclude属性来自定义排除的资源。例如，要仅排除/static，/public可以设置以下属性：
``spring.devtools.restart.exclude=static/**,public/**``

#### pom文件引入devtools依赖
```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<optional>true</optional>
		</dependency>
```
##### optional=true,依赖不会传递
#### maven插件配置
```
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<fork>true</fork>
				</configuration>
			</plugin>
```
#### 开启自动编译
设置里compiler勾选Build Project automatically
#### 开启运行时自动编译
command+shift+alt+/ 打开registry勾选compiler.automake.allow.when.app.running
#### 重新启动即配置完成
#### 更改代码后不需手动重启 build后即自动重启
#### 禁用
``spring.devtools.restart.enabled=false``
#### 监听其他路径
``spring.devtools.restart.additional-paths=``