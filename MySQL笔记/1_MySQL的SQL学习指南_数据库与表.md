## 1_MySQL的SQL学习指南_数据库与表



### 数据库相关操作

指定某个数据库是操作表的前提。

```mysql
#查看MySQL管理的数据库
show databases;
#指定具体的某个数据库
use database_name;
#创建数据库
create database database_name;
#查看创建的数据库的相关信息
show create database database_name \G;
#删除某个数据库
drop database database_name;
```

---



### 表相关操作

表是存储数据的基本单位

---



#### 创建、查看、删除表

```mysql
#查看当前数据库的所有表
show tables;
#创建表
create table table_name(
字段1 数据类型 (约束) (默认值),
字段2 数据类型 (约束) (默认值),
...
（约束）
);
#查看表基本结构
describe table_name;
desc table_name;
#查看表详细结构
show create table table_name \G;
#删除表
#删除被其它表关联的表需要先删除子表的外键
drop table table_name;
```

---



#### 修改表

```mysql
#修改表名
alter table table_name rename new_name;
#修改字段数据类型
alter table table_name modify 字段 数据类型;
#修改字段名
alter table table_name change 旧字段 新字段 数据类型;
#添加字段
#默认添加到最后一列
#first表示在表的最前面加上新字段
#after 表示在表内已有的一个字段后面加上新字段
alter table table_name add 字段1 数据类型 [约束][first][after 字段2];
#删除字段
alter table table_name drop 字段名;
#修改字段排列位置
alter table table_name 字段1 数据类型 [first][after 字段2];
#更改表的引擎
#InnoDB(default),MyISAM...
alter table table_name engine=引擎名;
#删除表的外键约束
alter table table_name drop foreign key 外键约束名;
```

---



#### 约束

##### 单主键约束

```mysql
#type 1
create table student(
sname varchar(10) primary key,
...
);

#type 2
create table student(
sname varchar(10),
...
(constraint constraint_name) primary key(sname) 
);
```

##### 多主键联合约束

```mysql
#type 1
create table student(
sname varchar(10) primary key,
...
);

#type 2
create table student(
sname varchar(10),
...
[constraint cons_name] primary key(sname) 
);
```

##### 外键约束

```mysql
#外键
#两表之间连接，可以是一列或者是多列，一张表可以有多个外键，
#对应的是另外一张表的主键，且数据类型必须匹配
[constraint cons_name] foreign key (字段1,...) references super_table_name(字段1,...);
```

##### 非空约束

```mysql
create table student(
sname varchar(10) not null,
...
);
```

##### 唯一性约束

```mysql
#type 1
create table student(
id varchar(8) primary key,
sname varchar(10) unique,
...
);
#type 2
create table student(
id varchar(8) primary key,
sname varchar(10),
...
[constraint cons_name] unique(sname)
);
```

##### 默认值约束

```mysql
create table student(
sname varchar(10) primary key,
credit smallint default 0,
...
);
```

##### 字段自增长

```mysql
#自增长字段
#默认起始值为1，一张表只能有一个自增长字段，且必须为主键的一部分，
#类型支持所有的整数类型
create table student(
id smallint primary key auto_increment,
sname varchar(10),
credit smallint default 0,
...
);
```

##### check约束

```mysql
#添加简单的附加约束
#1
create table student(
sname varchar(10),
credit smallint default 0,
sex char(1), 
...
[constraint cons_name] check(sex in ('f', 'm'))
);

#2
create table student(
sname varchar(10),
credit smallint default 0,
age smallint,
sex char(1), 
...
[constraint cons_name] check(age > 18)
);
```

