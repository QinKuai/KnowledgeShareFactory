## MySQL基础知识

### 0. 数据类型

#### 0.1 串数据类型

```mysql
# 串数据类型，即字符串
## 串数据类型都需要通过引号，通常为单引号括起来
char -- 1~255个字符的定长串，创建时指定
varchar -- 长度可变，最多不超过255字节，创建时指定最大字符长度

enum -- 接收最多64K个串组成的集合的某个串
set -- 接收最多64K个串组成的集合的零个或多个串

tinytext -- 与text相同，最大长度为255
mediumtext -- 与text相同，最大长度为16K
text -- 最大长度为64K的变长文本
longtext -- 与text相同，最大长度为4G 


# 定长串
## 不允许多余指定长度的字符数目
## 占据的内存空间与指定时相同且不变化

# 变长串
## 存储可变长度的文本
## 有些数据类型有最大定长，而有些则是完全变长的

# 为什么还有定长串？
## 处理定长串的速度会比处理变长串快
## 同时变长串不能被索引也会影响检索性能
```

#### 0.2 数值数据类型

```mysql
# 数值数据类型
## MySQL支持多种数据类型
## 支持的取值范围越大，所需存储空间也就越多
## 对于无符号数据只需要在数据类型前加上unsigned就行
## 数值类型不需要扩在引号中
bit -- 位字段，1~64位
boolean/bool -- 值为0或者1

tinyint -- 8位整数
smallint -- 16位整数
mediumint -- 24位整数
int/integer -- 32位整数
bigint -- 64位整数

float -- 单精度浮点
double -- 双精度浮点
decimal -- 精度可变的浮点
## MySQL没有专门的货币数据类型，一般使用decimal(8,2)
## 保留两位小数，后续小数位遵循四舍五入
```

#### 0.3 日期和时间数据类型

```mysql
date -- 能表示1000-01-01~9999-12-31之间的日期，格式位YYYY-MM—DD
time -- 格式为HH:MM:SS
datetime -- date和time的组合
timestamp -- 与datetime类似，但范围小一些
year -- 用两位数字表示，范围为70~69（1970~2069）；用四位数字表示，范围为1901-2155
```

#### 0.4 二进制数据类型

```mysql
tinyblob -- blob最大长度为255B
blob -- blob最大长度为64KB
mediumblob -- blob最大长度为16MB
longblob -- blob最大长度为4GB
```



### 1.状态检查

```mysql
# mysql状态检查
show status;

# 查看数据库或者表的创建sql语句
show create tableName \G
show create databaseName \G

# 查看账号授权状态
show grants;

# 查看一张表的具体结构
show columns tableName;
desc table_name;
```

 

### 2. SELECT

#### 2.1 基本

```mysql
# 查找一个字段的所有行
select colName from tableName;

# 查找多个字段的所有行
select colName1,colName2 from tableName;

# 查找所有字段所有行
select * from tableName;

# 查找字段的不重复行
# distinct不只作用于单字段上
# 如果查找多字段，会作用到所有字段上
select distinct colName from tableName;

# 限制查找出的数量
select colName from tableName limit Amount;
# 限制查找从哪列开始的固定数量的行
# Mysql第一行的index为0
select colName from tableName limit StartIndex, Amount;
select colName from tableName limit Amount offset StartIndex;
```

#### 2.2 排序

```mysql
# 前提：在未指定排序规则之前，不能认为mysql输出的顺序有意义
# 设置：关于大小写问题，取决于具体的数据库设置，默认不区分大小写

# 根据某列排序
# 选择用以排序的列可以是检索的列，也可以是未检索的列
# 默认升序排列
select * from tableName order by colName;

# 根据多列排序
# 多列排序的规则为
# 先排序colName1，若colName1相同，在根据colName2排序
# 默认升序排序
select * from tableName order by colName1, colName2;

# 更改为降序
## 单列
select * from tableName order by colName desc;
## 多列中某一列
select * from tableName order by colName1 desc, colName2;
## 多列中所有列
select * from tableName order by colName1 desc, colName2 desc;
```

####  2.3 定值过滤

> 为什么需要数据库过滤，而不是交给应用层？
>
> 1）通常应用层与数据库之间需要网络传输 ，传输数据需要时间，数据库过滤能减少数据量，缩短传输时间。
>
> 2）同时数据库能对过滤进行优化，提升过滤的效率。

```mysql
# where子句的位置应当在order by子句之前

# 简单过滤
## =, <, >, <=, >=, <>, != 用法类似
select * from tableName where colName=value1;
## 取值区间
select * from tableName where colName between value1 and value2;
## 空值检查
## null不同于0，空字符串或空格
## 使用= null并不能匹配到值为null的行
select * from tableName where colName is null;

# 组合过滤
## and 交集
select * from tableName where colName1=value1 and colName2=value2;
## or 并集
select * from tableName where colName1=value1 or colName2=value2;
## SQL默认先处理and，然后处理or，使用()可以更改处理顺序
## 通常出现多条件时都应使用()
select * from tableName where (colName1=value1 or colName2=value2) and colName3=value3;

# 给定目标集
select * from tableName where colName1 in (value1, value2);

# 取反
## MySQL中只支持not对in,between,exists使用
select * from tableName where cloName1 not in (value1, value2);
```

#### 2.4 通配符过滤

```mysql
# like操作符
# 通配符通常是有代价的
# 能不使用通配符就不用，实在要用就放在过滤条件的最后

# 百分号(%)通配符
# 表示在搜索串中任何字符出现任何次
## 例如检索app开头的词就能如下所示
select * from tableName1 where colName1 like 'app%';
## 通配符可以多个一起使用
## 查询包含a的词，包括a本身
select * from tableName1 where colName1 like '%a%';
## 也能放在中间
## 查询开头为s结尾为e的词，包括se
select * from tableName1 where colName1 like 's%e';
## 通配符不能匹配到空值，也就是null

# 下划线(_)通配符
# 表示在搜索串中只匹配一个字符
## son之前有且只有一个字符时会被匹配到
select * from tableName1 where colName1 like '_son';
```

#### 2.5 正则表达式过滤

```mysql
# 正则
# 正则表达式有正则表达式语言组成
# 建议有一定的正则基础


```

#### 2.6  计算字段

```mysql
# 将需要在应用层完成的工作交由处理速度更快的数据库完成

# 字段拼接
## concat()函数
## 其他DBMS可能使用+或者||来完成，转移时务必注意
select concat(colName1, ',', colName2) from tableName1;

## rtrim()函数
## 删除值右边的所有空格
## ltrim()函数 
## 删除值左边的所有空格
## trim()
## 删除值左右两边的所有空格
select rtrim(colName1) from tableName1;

# 字段别名
## as 关键字
## 在 外部调用 或者 解决歧义 时很重要
select concat(colName1, ',', colName2) as result from tableName1;

# 数学计算
## MySQL支持 +、-、*、/四种运算
## 同时可以通过()实现定义运算次序
select colName1 * colName2 from tableName1;
select (colName1 + colName2) * colName3 from tableName1;
```

#### 2.7 数据处理函数

```mysql
# 函数
## 可以通过mysql客户端的help查看函数具体说明
## 函数对于SQL的移植性并不好，因为每一个DBMS都有自己独特的函数实现

# 文本处理函数
## 大写 upper(text)
## 小写 lower(text)
## 字符串长度 length(text)
## 删除值右边的所有空格 rtrim(text)
## 删除值左边的所有空格 ltrim(text)
## 删除值左右两边的所有空格 trim(text)
## length的值大于字符串总长时，返回整个字符串
## 小于0时返回空字符串
## 左子串 left(text, length)
## 右子串 right(text, length)
## 该函数中字符串起始值为1
## 若startPos为-1则从字符串末尾开始计数
## 子串 substring(text, startPos) substring(text, startPos, length)

# 日期与时间处理函数
## 当前日期 curdate()
## 当前时间 curtime()
## 当前日期和时间 now()
## 返回日期和时间的日期部分 date(time)
## 返回日期和时间的日部分 day(time)
## 返回日期和时间的月部分 month(time) 
## 返回日期和时间的年部分 year(time)
## 返回日期和时间的小时部分 hour(time) 
## 返回日期和时间的分钟部分 minute(time)
## 返回日期和时间的秒部分 second(time)
## 固定格式输出日期时间 date_format(time, format)
## 当前日期是星期几(1为星期天以此类推) dayofweek(time)

## 在仅使用日期作为检索条件时
## 该语句存在一定的问题，因为在数据库中时间列的数据类型通常为datetime
## 2020-07-12后面默认时间时00:00:00，因此做检索时日期对了，时间不对也不行
select * from tableName1 where colName1 = '2020-07-12';
## 因此最好使用日期处理函数date()
select * from tableName1 where date(colName1) = '2020-07-12';

## 在检索一定给定月的数据时，有两种方案
## 给定日期范围
select * from tableName1 where date(colName1) between '2020-07-01' and '2020-07-31';
## 更优的方案是使用时间处理函数
select * from tableName1 where year(colName1) = 2020 and month(colName1) = 7;

# 数值处理函数
## 绝对值 abs(num)
## 圆周率 pi()
## 开方 sqrt(num)
## 随机数 rand()
## 取余 mod(num1, num2)
## 正弦 sin(num)
## 余弦 cos(num)
## 正切 tan(num)
```

#### 2.8 数据汇总

```mysql
# 平均值 avg
## avg函数忽略值为NULL的行
select avg(colName1) from tableName1 where colName1 = value1;
## avg只能用于确定特定数值列的平均值
## 多列的平均值需要多次使用avg

# 计数 count
## 使用count(*)计数所有的行，与行的具体值无关
## 使用count(colName1)计数具体行，忽略值为NULL的行
select count(*) from tableName1;
select count(colName1) from tableName1;

# 最大值 max
# 最小值 min
## 使用max,min时必须指定列,能返回数值或者日期的极值
## max,min会忽略值为NULL的行
select max(colName1), min(colName1) from tableName1;

# 总和 sum
## sum允许利用算数计算符，实现多列共同计算
## sum会忽略值为NULL的行
select sum(colName1*colName2) from tableName1;
```

#### 2.9 数据分组与过滤

```mysql
# group by分组
## group by应当使用在where之后、order by之前
## group by可以包含多个列，实现更精确的分组
## select后的非聚集函数列，都必须出现在group by后
select colName1, colName2, count(*) from tableName1 group by colName1, colName2;

# having过滤
## having与where相当类似，where中能使用的操作符having都能使用
## where能过滤行数据，使其不进入分组的步骤
## having则是过滤分组后的数据，使其不作为结果输出
select colName1 from tableName1 
where colName2 = value1 
group by colName1 
having count(*) > 2;
```

#### 2.10 SELECT总结

```mysql
# SELECT 语句的结构
SELECT (需要的列或是表达式)
FROM (检索的表名)
WHERE (行级过滤)
GROUP BY (分组说明)
HAVING (组级过滤)
ORDER BY (输出排序顺序)
LIMIT (输出的行数)
```



### 3. 子查询

#### 3.1 基本

```mysql
# 使用子查询配合IN实现过滤
select colName1 
from tableName1 
where colName2 in (select colName2 
                   from tableName2
                   where colName3 = value1);
## 子查询通过嵌套select语句实现
## 但同时多层嵌套会带来性能上的问题

# 作为计算字段使用子查询
select colName1,
		(select count(*)
        from tableName2
        where tableName2.colName2=tableName1.colName2) as name1
from tableName1
order by colName1;
```



### 4. 联结

#### 4.1 基本

```mysql
# 联结 join
## 联结在实际的数据库表中不存在
## 仅存在于查询的执行中

# 笛卡尔积
## 没有联结条件的表关系返回的结果为笛卡儿积
## 检索出来的行的数目将是所有表中的行数的积

# 创建联结
## 等值联结(equijoin)又称内部联结(innerjoin)
## 首推后者，更直接明显且可能提升性能
select * 
from tableName1, tableName2
where tableName1.colName1 = tableName2.colName1;

select * 
from tableName1 inner join tableName2
on tableName1.colName1 = tableName2.colName1;

## 以下两种方式结果相同，但实现不同
## 具体如何选择，应根据具体情况测试得到
select colName1 
from tableName1 
where colName2 in (select colName2 
                   from tableName2
                   where colName3 = value1);
select colName1 
from tableName1 inner join tableName2
on tableName1.colName2 = tableName2.colName2
where colName3 = value1;                   
```

#### 4.2 高级联结

```mysql
# 表别名
## SQL允许为列名、计算字段以及表起别名
## 为表起别名主要是为了能在一个select语句中多次使用同一张表
select * 
from tableName1 as a, tableName2 as b
where a.colName1 = b.colName1
and a.colName2 = value1;

# 自联结
## 多次使用同一张表的体现
## 自联结能用来替代在相同表中检索数据时使用的子查询
## 子查询
select colName1, colName2
from tableName1
where colName3 = (select colName3
                 fromm tableName1
                 where colName1 = value1);
## 自联结
select t1.colName1, t2.colName2
from tableName1 as t1, tableName1 as t2
where t1.colName3 = t2.colName3
and t2.colName3 = value1;
                 
# 自然联结
## 只能选择那些唯一的列
## 一般不会遇到不是自然联结的内部联结
select t1.*, t2.colName1
from tableName1 as t1, tableName as t2
where t1.colName2 = t2.colName2;

# 外部联结
## 有时需要关联表中没有关联行的那些行
## 也即一个表中存在该值，但另一表中暂时没有前表中的某些行的数据
## 但在做数据检索时需要这些行，这就是外部联结
select tableName1.colName1, tableName2.colName2
from tableName1 left outer join tableName2
on tableName1.colName3 = tableName2.colName3;
## 除了left join还有right join，前者选取所有行的表是前边的表，后者选取所有行的表是后边的表
## 同时可以通过变化from后的表顺序实现同样的效果


# 带聚集函数的联结
select tableName1.colName1,
		tableName1.colName2,
		count(tableName2.colName3)
from tableName1 inner join tableName2
on tableName1.colName2 = tableName2.colName2
group by tableName1.colName2;

# 使用联结时需要注意的点
## 1. 一般使用内部联结，也会使用到外部联结
## 2. 总是使用连接条件，避免出现输出表的笛卡尔积
```



### 5. 组合查询

#### 5.1 基本

```mysql
# 组合查询
## 一般SQL查询只包含从一张或多张表中返回的数据的单条select语句
## 组合查询实现多条select语句，并将结果作为一个结果集返回

# 何时使用组合查询？
## 1.单个查询中从不同表中返回相似结构的数据
## 2.对单个表执行多次查询，按单个查询返回数据

# 组合查询与多where条件具有相似性，能互相转化，可能会出现性能差异

# UNION 组合/取并集
## 每个selct之间必须通过union隔开
## 每个select都必须包含完全相同的列、表达式和聚集函数或者列数据类型兼容
select colName1, colName2
from tableName1
where colName1 = value1
union
select colName1, colName2
from tableName1
where colName2 = value2;
## 若要实现并集不去重的效果，就需要使用union all，而这方面where并不能实现

# 对组合结果排序
## 使用order by实现，只能放在最后一条select语句之后，且只能出现一条
select colName1, colName2
from tableName1
where colName1 = value1
union
select colName1, colName2
from tableName1
where colName2 = value2
order by colName1;
```



### 6. 全文本检索

```mysql
# 并不是所有MySQL引擎都支持全文本检索
## 常用的InnoDB不支持，MyISAM支持
```



### 7. INSERT

#### 7.1 基本

```mysql
# 插入语句
## 插入语句一般不会产生输出

## 插入完成的行数据
insert into tableName1 values(value1, value2, ...);
## 插入指定列的行数据
## 对于一个表而言，省略列插入需要满足以下条件中的一个
## 1)该列的值允许为NULL
## 2)该列给定了默认值
insert into tableName1(colName1, colName2) values(value1, value2);

## 对于检索更为重要的数据库而言，尤其是对带索引的表插入、更新、删除都可能耗时
## insert 以及后续的update、delete都能使用low_priority来降低优先级，提升select的检索性能
insert low_priority into tableName1(...) values(...);

# 多行插入
insert into tableName1(colName1, colName2) values(value1, value2),
											(value3, value4),
											(value5, value6);
											
# 插入搜索的数据
## insert select
## select后的列名不需要和insert后的表列名匹配
insert into tableName2(colName1, colName2) select colName1, colName3 from tableName1;
```



### 8. UPDATE & DELETE

#### 8.1 UPDATE

```mysql
# update
## 基本语句
update tableName1 set colName1 = value1 where colName2=value2;
## 更新多列
update tableName1 set colName1 = value1, colName2 = value2 where colName3=value3;
## 子查询更新
update tableName1 
set colName1 = (select colName2 
                from tableName2 
                where colName3=value2)
where colName4 = value3;
## MySQL不支持在需要更新的表中使用子查询
## 因此需要通过在子查询外包一层select实现
update tableName1
set colName1 = (select *
               from (select colName1
                    from tableName1
                    where colName2 = value1) as tb1)
where colName2=value2;

## 使用update更新多行时
## 可能会出现更新到一定数量的行后
## 某一行报错退出
## 这时默认时舍弃之前的更新，将表返还更新前的状态
## 但可以通过ignore更改该设置
update ignore tableName1 ...
```

#### 8.2 DELETE

```mysql
# delete
## 基本语句
## delete实现的时删除表中的某一个或是某些个行
## 并不能删除表本身
delete from tableName1 where colName1=value1;
## 若要删除表内所有数据
## 使用truncate table会更快一些
truncate table tableName1;
```



### 9. 创建和操纵表

#### 9.1 创建

```mysql
# create table
## 基本语句
create table tableName1(
colName1 dataType1 auto_increment,
colName2 dataType2 not null,
colName3 dataType3 null default value1,
primary key(colName1)
) engine=InnoDB;
## 当表已在数据库中存在时，该语句执行失败
## 如果要在执行时确认表是否已存在来决定是否创建
create table if not exists tableName1 ...;

# null值
## 列设置默认为null
## 某列设置为null表示，该列值可以为空，即插入、更新时不必给定值
## 某列设置为not null表示，该列值不能为空，即插入、更新时必须给定值

# 主键
## 主键可以为一列或者多列
## 主键必须为为not null
primary key(colName1)
primary key(colName1, colName2)

# auto_increment
## 每个表只允许一个自增列，且必须被索引，例如使其成为主键
## 要获取insert后该列的自增长列的值
## 可以通过select last_insert_id()实现

# 默认值
## 通过default配置默认值
## MySQL只允许使用常量作为默认值，不允许使用函数

# 引擎
## MySQL内部打包了多种引擎，供不同场景使用
## 以下为常用引擎：
## 1.InnoDB：可靠的事务处理引擎，不支持全文本检索
## 2.MEMORY：功能等同于MyISAM，但数据存储与内存中，速度快，适合临时表
## 3.MyISAM：性能好，支持全文本检索但不支持事务处理
## 引擎之间不能使用外键，也就是外键不能跨引擎
```

#### 9.2 更改

```mysql
# alter table
## 理论上表在创建完成后，就不应该被更新了
## 基本语句
alter table tableName1 ...;

# 例如
## 增加列
alter table tableName1 add colName1 dataType not null;
## 删除列
alter table tableName1 drop column colName1;
## 外键
alter table tableName1 add constraint fk_name
foreign key (colName1) references tableName2(colName1);

## alter table应当谨慎使用，因为其执行后不可恢复
```

#### 9.3 删除&重命名

```mysql
# drop table
## 删除整张表
drop table tableName1;

# rename
## 重命名表
rename table tableName1 to tableName2;
## 重命名多表
rename table tableName1 to tableName2, tableName3 to tableName4;
```



### 10. 视图

#### 10.1 基本

```mysql
# 视图
## 视图是虚拟的表，只包含使用时动态检索数据的查询
create view viewName as
select ...;
```



### 11. 存储过程

#### 11.1 基本

```mysql
# 存储过程
## 将完成一个功能所需的一系列操作打包成一个存储过程，可以类比于函数
## 提供数据一致性和更好的安全性

# 使用存储过程
call procedureName1();

# 创建存储过程
create procedure procedureName1()
begin
	select ...;
end;
## 特别注意，如果使用mysql命令行则需要通过delimiter替换换行符
delimiter //
create procedure procedureName1()
begin
	select ...;
end //
delimiter ;

# 删除存储过程
## 加上if exists可以判断存储过程是否存在
## 一次避免存储过程不存在时，删除命令报错
drop procedure [if exists] procedureName1;

# 查看存储过程实现
show create procedure procedureName1 \G;
# 查看存储过程详细信息
show procedure status like '...';
```

#### 11.2 使用参数

```mysql
# 在存储过程中带上参数
## 一般参数分为两种，输入参数和输出结果参数
## 输入参数可直接使用，输出参数可以通过select into赋值
create procedure procedureName1(
	in param1 int,
    out param2 varchar(50)
)
begin
	select colName1
	from tableName1
	where colName2=param1
	into param2;
end;

## 调用含参数的存储过程
call procedureName1(100, @output);
## output作为临时变量，可以通过select获取
select @output;
```

#### 11.3 存储过程中局部变量

```mysql
# 在存储过程中可以定义局部变量，以实现更复杂的逻辑
# 还能使用if做逻辑控制

create procedure procedureName1(
	in param1 int,
    out param2 varchar(50)
)
begin
	declare temp1 int default 0;

	select colName1
	from tableName1
	where colName2=param1
	into param2;
	
	if temp1 == 0 then
    	select 1 into temp1;
    end if;
	
end;
```



### 12.游标

#### 12.1 基本

```mysql
# cursor
## 游标是一个存储在MySQL服务器上的数据库查询
## 不是一条select语句，而是被语句检索出来的结果集
## MySQL的游标只能作用于存储过程

## 基本结构
## declare的定义顺序是有规定的
## 一般是变量 -> 游标 -> 句柄
create procedure procedureName()
begin
	-- 变量的声明必须在游标和句柄之前定义
	declare o dateType1;
	
	declare cursorName1 cursor
	for
	-- 这里设置游标对应的select逻辑
	select colName1 from tableName1;
	
	-- 句柄定义必须在游标之后定义
	-- 为循环结束做设定
	declare continue handler for sqlstate '02000' set done=1;
	
	-- 开启游标
	open cursorName1;
	
	-- 一切与游标相关的操作都在此处完成
	-- 赋值，取当前的首行
	fetch cursorName1 into o;
	
	-- 循环
	repeat
		fetch cursorName1 into o;
		-- ...
	until done end repeat;
	
	-- 结束游标
	close cursorName1;
end
```



### 13. 触发器

#### 13.1 基本

```mysql
# trigger
## 触发器是能响应增删改语句并自动执行的语句
## 基本结构
## 触发器针对insert、update、delete三种行为
## 能在规定触发器在该行为前触发(before)，还是后触发(after)
## MySQL的触发器暂不支持call，也就是无法在触发器中使用存储过程
create trigger triggerName1 after insert on tableName1
for each row
begin
	-- ...
end

# 触发器的执行
## 如果before触发器执行失败，则语句不会执行，同时after触发器也不会执行
## 若语句执行失败，则after触发器不会执行

## MySQL的触发器是行级触发器
## 因此对于一个带来多行受影响的SQL语句，如删除多行、插入多行
## 对应触发器会相应地执行多次

# 删除触发器
## 触发器无法覆盖，即不能修改
drop trigger triggerName;

# insert触发器
## insert触发器代码中，可引用一个虚拟表NEW，访问被插入的行
## before insert触发器中，new中的值可以被更新
## 对于自增列，before触发器获取到的值为0，after获取到值为MySQL生成的自增值
## 这里可以使用来实现获取插入后的自增列值
create trigger triggerName1 after insert on tableName1
for each row select NEW.colName1 as autoIncrementCol;

# delete触发器
## delete触发器代码内，可引用一个虚拟表OLD，访问被删除的行，该表是只读的
## 使用before delete的好处很明显
## 如果在备份删除不成功时，删除动作将被放弃
create trigger triggerName2 before delete on tableName1
for each row
begin
	insert into tableName2(colName1) values(OLD.colName2);
end

# update 触发器
## update触发器代码中，可引用虚拟表NEW、OLD
## NEW表在before update中可更新，OLD表始终为只读
create trigger triggerName3 before update on tableName1
for each row
begin
	set NEW.colName1 = upper(NEW.colName1);
end
```



### 14. 事务管理

#### 14.1 基本

```mysql
# transaction
## 事务就是将一系列操作打包，保证这些操作要么都成功执行，要么都不执行

# 明确的事务管理的支持与引擎相关
## MyISAM并没有明确支持事务管理
## InnoDB则支持

# 事务相关关键词
## 事务 transaction：一组SQL语句
## 回退 rollback：撤销指定的SQL语句的过程
## 提交 commit：将未储存的SQL结果写入数据库表中
## 保留点 savepoint 事务处理中的临时占位符，可以对该点发布回退

# 事务开始
## 事务回退或是提交后，事务会自动关闭
start transaction;
# 事务回退
## 只能回退insert、update、delete，对于create和drop并不能回退
rollback;
# 事务提交
commit;
# 事务保留点
## 通过保留点能实现部分事务管理，能准确回退到指定保留点
## 以实现更灵活的事务控制
savepoint savepointName1;
# 回退到指定保留点
rollback to savepointName1;

# MySQL默认自动提交语句造成的更改
## 可以通过设置实现不自动提交运行后语句的更改
## 通过手动commit提交更改
## 该配置针对连接，而不针对数据库
## 也就说每次新的数据库连接默认都是自动提交的
set autocommit=0;
```



### 15. 字符集与语言

#### 15.1 基本

```mysql
# 基本概念
## 字符集：字母和符号的集合
## 编码：某个字符在字符集成员的内部表示
## 校对：规定字符如何比较的指令

# 查看支持的字符集
show character set;
# 查看支持的校对列表
## 小tip：区分大小写尾部为_cs，不区分大小写尾部为_ci
show collation;

# 查看数据库默认字符集
show variables like 'character%';
# 查看数据库默认校对
show variables like 'collation%';
```

#### 15.2 设置字符集与校对

```mysql
# 字符集很少是服务器或是数据库范围的设置
## 不同的表或是不同的列都可能采用不同的字符集
## 上述两种都能在创建表的时候指定

# 表级
## 若不指定字符集或是校对
## 默认使用系统默认值
create table tableName1(
	colName1 dataType,
    colName2 dataType
) default character set charsetName
collate collationName;

# 列级
create table tableName2(
	colName1 dataType,
    colName2 dataType character set charsetName1 collate collationName1
)default character set charsetName2
collate collationName2;

# 临时指定校对策略,会覆盖掉表或是列设置好的校对策略
select * from tableName3 order by colName1 collate collationName1;
```



### 16 安全管理

#### 16.1 基本

```mysql
# 安全管理
## 真实环境下最好不要使用root

# 管理用户
use mysql;
select user from user;

# 创建用户
## 不指定口令
create user username1
## 指定口令
create user username2 identified by 'password1';
# 修改用户名
rename user oldUsername to newUsername;
# 删除用户
drop user username1;
```

#### 16.2 设置访问权限

```mysql
# 查看用户的权限情况
show grants for username1;

# 设置权限
grant select,update on databaseName1.* to username1;
grant insert on datebaseName2.tableName1 to username1;
# 撤销权限
revoke select on databaseName1.* from username1;
```

#### 16.3 权限列表

```mysql

```



### 17. 数据库维护

#### 17.1 备份

```mysql
# 方案1
## 命令行使用mysqldump转存数据库到某个外部文件

# 方案2
## 命令行使用mysqlhotcopy从一个数据库复制所有数据
## 并非所有数据库引擎都支持该程序

# 方案3
## 使用backup table或select into outfile转存所有数据到外部文件
## 由于这两个命令要接收文件名，必须保证外部文件不存在，不然会报错

# 刷新尚未写到磁盘的数据，包括索引数据
flush tables;
```

