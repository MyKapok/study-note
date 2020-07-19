# Mybatis

## 框架简述

### 什么是框架

它是我们软件开发中的一种解决方案，不同的框架解决的是不同的问题

使用框架的好处：

* 框架封装了很多细节，使开发者可以使用极简的方式实现功能，大大提高开发效率。

### 三层架构

* 表现层：用于展示数据的
* 业务逻辑层：处理业务需求
* 持久层：和数据库交互的

### 持久层技术解决方案

* JDBC技术
* Spring的JDBCTemplate：Spring中对JDBC的简单封装
* Apache的DBUlits

以上都不是框架 

### mybatis 框架

是一个持久层框架，用Java编写；

封装了JDBC的许多细节，使开发者只需要关注sql语句本身，无需关注注册驱动，创建连接等过程；

采用ORM思想解决了实体和数据库映射的关系。

ORM：Object　Relational　Mapping 对象关系映射

* 创建实体类和DAO接口
* 创建Mybatis的主配置文件sqlmapconfig.xml
* 创建映射配置文件 IUserMapper.xml

环境搭建的注意事项:

1. mybatis的映射文件位置必须和dao接口的包结构相同
2. 映射配置文件的mapper标签namespace属性的取值必须是dao接口的全限定类名
3. 映射配置文件的操作配置,id属性的取值必须是dao接口的方法名

<img src="C:\Users\acer\AppData\Roaming\Typora\typora-user-images\image-20200712180329679.png" alt="image-20200712180329679" style="zoom:200%;" />

### 自定义mybatis的分析

## springboot +Mybatis













