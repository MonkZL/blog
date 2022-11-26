---
title: java-web学习 MySQL
date: 2022-11-20 17:16:53
tags: MySQL
categories: java-web
---

## 数据库相关概念

### 数据库（DB）
储存数据的仓库，数据是有组织的进行储存的

### 数据库管理系统（DBMS：DateBase Manager Server）
管理数据库的软件（Oracle，MySQL，sqlite等等都是数据库管理系统）

### SQL（Structured Queue Language）
* 结构化查询语言
* 操作关系型数据库的编程语言
* 定义操作所有类型数据库的统一标准

## 关系型数据库
通过表来储存数据的数据库

## SQL

### 简介
* 结构化查询语言
* 操作关系型数据库的编程语言
* 定义操作所有类型数据库的统一标准
* 不同数据库会存在不同用法（方言）

### 通用语法
+ 可以单行多行书写，以分号（;）结尾
+ 语句不区分大小写
+ 注释
  + 单行注释：```-- 注释内容``` 或 ```#注释内容```（MySQL特有）
  + 多行注释：```/* 注释 */```

### SQL分类

#### DDL（Data Definition Language）数据定义语言 操作数据库，表等
+ show databases; // 查看所有数据库
+ create database db1; // 创建一个名字为db1的数据库
+ create database if not exists db1; // 创建一个名字为db1的数据库，如果它不存在的话
+ drop database db1; // 删除一个名字为db1的数据库
+ drop database if exists db1; // 删除一个名字为db1的数据库，如果它存在的话
+ use db1; // 使用db1
+ select database(); // 查看当前使用的数据库
+ show tables; // 查看所有表
+ desc user; // 查看user表的结构
+ create table user(id int,username varchar(20),password varchar(32)); // 创建一个名字为table1的表
+ drop table user; // 删除一个名字位user的表
+ drop table if exists user; // 删除一个名字位user的表，如果它存在的话
+ alter table user rename to tb_user; // 修改user表的名字为tb_user
+ alter table user add age int; // 给user表新增一列（age）
+ alert table user modify name varchar(20); // 修改列的类型
+ alter table user change name username varchar(20); // 修改列的名字和类型 name改为username 类型改为varchar(20)
+ alter table user drop username; // 删除列

##### 常用的数据类型
+ int // age int
+ double // score double(总长度,保留小数点后多少位) 比如0～100保留2位小数 double(5,2)
+ date // birthday date
+ char // 定长字符串 name char(10) 存"张三" 占用10个字符空间 储存性能高 浪费空间
+ varchar// 变长字符串 name varchar(10) 存"张三" 占用2个字符空间 储存性能低 节约空间

#### DML（Data Manipulation Language）数据操作语言 对表中的数据进行增删改
+ insert into user(id,name) values(1,"zl"); // 给指定列插入数据
+ insert into user values(1,"zl"); // 给所有列添加数据，列名的列表是可以省略的
+ insert into user values(1,"zl"),(1,"zl"),(1,"zl"),(1,"zl"); // 插入多行
+ update user set username = "zs" where username = "zl"; // 把username为zl的username修改为zs
+ update user set username = "zs"; // 没有where条件判断 会修改所有数据
+ delete from user where username = "zl"; // 删除username为zl的数据
+ delete from user; // 没有where条件判断 会删除所有数据

#### DQL（Data Query Language）数据查询语言 对表中的数据进行查询
select 字段名称 from 表名 where 条件列表 group by 分支字段 having 分组后的条件 order by 排序字段 limit 分页限定
##### 基础查询
+ select * from user; // 查询user的所有数据
+ select username from user; // 查询user的username列
+ select distinct address from user; // 去重查询
+ select username as 名字 from user; // 给列取别名

##### 条件查询
+ select * from user where age = 45; // 年龄45的
+ select * from user where age != 45; // 年龄不为45的
+ select * from user where age <> 45; // 年龄不为45的
+ select * from user where age >= 45; // 年龄大于等于45的
+ select * from user where age > 45 && age < 60; // 年龄在45到60之间的
+ select * from user where age > 45 and age < 60; // 年龄在45到60之间的
+ select * from user where age between 45 and 60; // 年龄在45到60之间的（边界要取）
+ select * from user where age = 45 or age = 60; // 年龄是45或者60
+ select * from user where age = 45 || age = 60; // 年龄是45或者60
+ select * from user where age in (45,60); // 年龄是45或者60
+ select * from user where english is null; // 查询english为null的数据
+ select * from user where english is not null; // 查询english不为null的数据
+ select * from user where name like "马%"; // 查询姓马的的人 %匹配多个字符
+ select * from user where name like "马%疼"; // 查询名字为马*疼的人
+ select * from user where name like "_花%"; // 查询第二个字是花的人 _匹配单个字符

##### 排序查询
+ select * from user order by age asc; // 升序排列
+ select * from user order by age desc; // 降序排列
+ select * from user order by math desc , english desc; // 多个排列条件

##### 聚合函数
+ select count(id) from user; // count统计数量，列不为null都会纳入计算
+ select max(id) from user; // 最大，列不为null都会纳入计算
+ select min(id) from user; // 最小，列不为null都会纳入计算
+ select sum(id) from user; // 总和，列不为null都会纳入计算
+ select avg(id) from user; // 平均，列不为null都会纳入计算
+ todo 其他函数

##### 分组查询
+ select sex, avg(math) from user group by sex; // 查询不同性别数学的平均分
+ select sex, avg(math), count(*) from user group by sex; // 查询不同性别数学的平均分以及各自人数
+ select sex, avg(math), count(*) from user where math > 90 group by sex; // 查询不同性别数学的平均分以及各自人数，要求分数低于90的算到分组内
+ select sex, avg(math), count(*) from user where math > 90 group by sex having avg(math) > 95; // 查询不同性别数学的平均分以及各自人数，要求分数低于90的算到分组内，且分组后平均分大于95的

##### 分页查询
+ select * from user limit 1,3; // limit start,length 从2个开始查3个数
+ select * from user limit 3; // 从第一个开始 查3个数

#### DCL（Data Control Language）数据控制语言 对数据进行权限控制
todo


### 约束

#### 概念
+ 约束是作用与列上的规则，用于限制加入表的数据
+ 约束的存在保证了数据库中数据的正确性，有效性和完整性

#### 约束的分类
+ 非空约束（NOT NULL）
+ 唯一约束（UNIQUE）
+ 主键约束（PRIMARY KEY）
+ 检查约束（CHECK）// mysql不支持该项
+ 默认约束（DEFAULT）

```
create table user (
  id INT PRIMARY KEY auto_increment, // 主键
  ename VARCHAR(10) NOT NULL UNIQUE, // 非空 唯一
  joindata DATA NOT NULL,
  salary DOUBLE(7,2) NOT NULL,
  bonus DOUBLE(7,2) DEFAULT 0 // 默认
);
```

+ 外键约束（FOREIGN KEY）

```
// 部门表
create table dep(
  id INT PRIMARY KEY auto_increment,
  name varchar(20)
);
// 员工表
create table emp(
  id INT PRIMARY KEY auto_increment,
  name varchar(20),
  dep_id int,
  constraint fk_emp_dep foreign key(dep_id) references dep(id) // 创建外键
);
将员工表的dep_id和部门表的id相关联
以后插入新员工的时候，dep_id必须是部门表已有的id
删除部门表的数据时，该部门表的id没有在员工表的dep_id中出现过
```

### 数据库设计

#### 一对多
```
// 部门表
create table dep(
  id INT PRIMARY KEY auto_increment,
  name varchar(20)
);
// 员工表
create table emp(
  id INT PRIMARY KEY auto_increment,
  name varchar(20),
  dep_id int,
  constraint fk_dep_id foreign key(dep_id) references dep(id) // 创建外键
);
```

#### 多对多
```
// 商品表
create table tb_goods(
  id INT PRIMARY KEY auto_increment,
);
// 订单表
create table tb_order(
  id INT PRIMARY KEY auto_increment,
);
// 商品和订单的中间表
create table tb_goods_order(
  goods_id INT,
  order_id INT,
  count INT,
  constraint fk_goods_id foreign key(goods_id) references tb_goods(id),
  constraint fk_order_id foreign key(order_id) references tb_order(id)
);
```

#### 一对一
```
// 用户表
create table tb_user(
  id INT PRIMARY KEY auto_increment,
  name varchar(10),
  detail_id INT UNIQUE,
  constraint fk_detail_id foreign key(detail_id) references tb_user_detail(id)
);
// 用户详情表
create table tb_user_detail(
  id INT PRIMARY KEY auto_increment,
  address varchar(20)
)  
```

### 多表查询

#### 连接查询
##### 内连接查询（查询的是A和B相交的部分）
###### 隐式内连接
```
SELECT 字段列表 FROM 表1,表2... where 条件
```
###### 显式内连接
```
SELECT 字段列表 FROM 表1 [INNER] join 表2 on 条件
```
##### 外连接查询
###### 左外连接查询（查询的是A所有和B相交的部分）
```
SELECT 字段列表 FROM 表1 LEFT [OUTER] join 表2 on 条件
```
###### 右外连接查询（查询的是B所有和A相交的部分）
```
SELECT 字段列表 FROM 表1 RIGHT [OUTER] join 表2 on 条件
```
#### 子查询
查询中嵌套查询

##### 子查根据查询结果的不同，作用不同

+ 单行单列：作为条件值，使用 = != > < 等进行条件判断
```
SELECT 字段列表 FROM 表 WHERE 字段名 = (子查询)
```
+ 多行单列：作为条件值，使用 in 等关键词进行条件判断
```
SELECT 字段列表 FROM 表 WHERE 字段名 in (子查询)
```
+ 多行多列：作为虚拟表
```
SELECT 字段列表 FROM (子查询) WHERE 条件
```
  




