## 4_MySQL的SQL学习指南_存储过程&函数



### 存储过程

```mysql
#创建存储过程
create procedure sp_name ([proc_para])
[characteristics] routine_body
```

- 1.`sp_name`:存储过程的名称

- 2.`proc_para`：指定存储过程的参数列表
  `[IN | OUT | INOUT] para_name type`
  `IN` 表示输入参数
  `OUT `表示输出参数
  `INOUT` 表示既可输入也可输出
  `type` 表示`MySQL`允许的数据类型

- 3.`characteristics`表示存储过程的特性:

  - `LANGUAGE SQL`:`SQL`是`LANGUAGE`特性的唯一值

  - `[NOT] DETERMINISTIC`:默认为`NOT`，表示相同的输入可能得到不同的输出

  - `{CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA}`:

    默认为`CONTAINS`。

    `CONTAINS`表示包含SQL语句，但不包含读写语句；`NO`表示不包含SQL；`READS`表示包含读的语句;`MODIFIES`表示包含写的语句。

  - `SQL SECURUTY {DEFINER | INVOKER}`表示权限定义：

    默认为`DEFINER`

    `DEFINER`表示定义者能够执行;`INVOKER`表示有权限者能执行。

  - `COMMENT 'string'`: 注释信息，用以描述

- 4.`routine_body`为SQL代码内容，使用BEGIN...END表示开始和结束。

#### 实际写存储过程

```mysql
#在MySQL中，写存储过程之前必须要修改结束符，不然无法正常运作,并在结束后修改回来

#不含参数的存储过程
delimiter //

create procedure showall()
begin
select * from sc;
end//

delimiter ;

#包含输出参数的存储过程
create procedure countScore(out param1 INT)
begin
select count(*) into param1 from sc;
end//



```





### 存储函数

```mysql
#创建存储函数
create function func_name([func_para])
returns type
[characteristic] routine_body
```

- `func_name`:存储函数名

- `func_para`:存储函数参数列表

  `[in | out | inout] para_name type`:这一部分与存储过程完全一致,但是函数中参数总是`in`

- `returns type`:表示返回数据的类型

- `characteristic`:与存储过程完全一致

#### 实际写存储函数

```mysql
#在写存储函数时，同样需要注意结束符的变更

delimiter //

create function showSname()
returns varchar(20)
reads sql data
return (select sname from s where s.sno='17301087');//

delimiter ;

```



### 变量的使用

**变量作用域一般在begin...end之间**

#### 1.定义变量：

```mysql
declare var_name[,var_name]... data_type [default value];
```

#### 2.赋值变量：

```mysql
set var_name = expr[, var_name=expr]...;
#通过select...into进行赋值
select col_name[,..] into var_name[,...] table_expr;
```

#### 3.定义条件和处理程序：

##### 定义条件

```mysql
#条件定义
declare condition_name condition for [condition_type]

[condition_type]:
sqlstate [value] sqlstate_value | mysql_error_code

1.condition_name:条件名
2.condition_type:条件类型
3.sqlstate_value | mysql_error_code:都表示MySQL的错误
	对于一个MySQL错误，例如ERROR 1142（42000）
	sqlstate_value=42000
	mysql_error_code=1142


#条件定义实例
declare command_not_allowed condition for sqlstate '42000';
or
declare command_not_allowed condition for 1148;
```

##### 处理程序

```mysql
#处理程序：
declare handler_type handler for condition_value[,...] sp_statement
handler_type:
	continue|exit|undo
	注:
	continue遇到错误不处理继续执行
	exit遇到错误推出
	undo遇到错误撤消之前的操作，MySQL不支持
	
	
condition_value:
	sqlstate [value] sqlstate_value
	|condition_name
	|sqlwarning 匹配所有01开头的sqlstate错误代码
	|not found 匹配所有02开头的sqlstate错误代码
	|sqlexception 匹配所有没别sqlwarnig和not found捕获的sqlstate错误代码
	|mysql_error_code
	
#实例
#1.捕获sqlstate_value
declare continue handler for sqlstate '42S02' set @info='NO_SUCH_TABLE';

#2.捕获mysql_state_code
declare continue handler for 1146 set @info='NO_SUCH_TABLE';

#3.先定义条件，然后调用
declare no_such_table condition for 1146;
declare continue handler for no_such_table set @info='...';

#4.使用sqlwarning
declare exit handler for sqlwarning set @info='WARNING';

#5.使用not found
declare exit handler for not found set @info='...';

#6.使用sqlexception
declare exit handler for sqlexception set @info='ERROR'
```

#### 4.光标

查询语句可能返回多条记录，需要在**存储过程或是函数**中使用光标来**逐条读取结果集中的记录**。

##### 声明光标：

```mysql
declare cursor_name cursor for select_statement
```

##### 打开光标：

```mysql
open cursor_name
```

##### 使用光标：

```mysql
fetch cursor_name into var_name[,var_name]...
#var_name必须是之前定义好的变量
```

##### 关闭光标：

```mysql
close cursor_name
#如果光标未被显式关闭，则会在声明的复合语句的末尾被关闭
```

#### 5.流程控制

##### if：

```mysql
if expr_condition then statement_list
[elseif expr_condition then statement_list]...
[else statement_list]
end if

#实例
if val is null
then select 'val is null';
else select 'val is not null';
end if
```

##### case：

```mysql
#type 1
case case_expr
when when_value then statement_expr;
[when when_value then statement_expr]...
[else statement_list]
end case

#实例
case var1
when 1 then select 'var1 is 1';
when 2 then select 'var1 is 2';
else select 'var1 is not 1 or 2';
end case

#type 2
case
when expr_condition then statement_list
[when expr_condition then statement_list]...
[else statement_list]
end case;

#实例
case
when val is null then select 'val is null';
when val < 0 then select 'val < 0';
when val > 0 then select 'val > 0';
else select 'val is 0';
end case;
```

##### loop：

```mysql
[loop_label:] loop
statement_list
end loop [loop_label]
#通过leave语句跳出循环

#实例
declare id int default 0;
add_loop: loop
	set id = id + 1;
	if id>=10 then leave add_loop;
	end if;
end loop add_loop;
```

##### leave：

```mysql
#能够推出任何被标记的流程控制构造
#通常可以和loop或者begin...end一起使用
leave label

#实例
add_num: loop
set @count=@count+1;
if @count=50 then leave add_num;
end loop add_num;
```

##### iterate:

```mysql
#iterate能将执行顺序跳转到开头处
#只能出现在loop、repeat或者while语句内
#类似于编程语言中的continue，直接终端当前循环，执行下一次循环
iterate label

#实例
declare p1 int default 0;
my_loop: loop
set p1=p1+1;
if p1<10 then iterate my_loop;
elseif p1>20 then leave my_loop;
end if;
end loop my_loop;
end;
```

##### repeat:

```mysql
#repeat能创建一个带条件的循环过程，如果为真则循环结束，为假则继续循环。
[repeat_label:]repeat
statement_list
until expr_condition
end repeat [repeat_label]

#实例
declare id int default 0;
repeat
set id = id +1;
until id>=10;
end repeat;
```

##### while:

```mysql
#while也能创建一个带条件的循环过程，如果条件为真则循环，为假则退出
[while_label:] while expr_condition do
statement_list
end while [while_lable]

#实例
declare i int default 0;
while i<10 do
set i=i+1;
end while;
```



### 调用存储过程和函数

#### 调用存储过程

```mysql
#调用当前数据库的存储过程
call procedure_name([para[,...]]);
#调用其他数据库的存储过程
call dbname.procedure_name([para[,...]]);
```

#### 调用函数

```mysql
#用户定义的函数实质上和系统定义的函数性质相同
#因此使用方法也是相同的
```



### 查看存储过程和函数

#### 1.通过show status查看

```mysql
show {procedure | function} status [like 'pattern'] \G
#like能匹配名称
#这个命令能查到所有的存储过程和函数，包括自定义和系统定义的
#\G能简化输出结果
```

#### 2.通过show create查看

```mysql
show create {procedure | function} sp_name
#sp_name表示存储过程或是函数的名称
#能获取到具体的代码内容
```

#### 3.通过从information_schema.routines表中查询

```mysql
select * from information_schema.routines
where routine_name = 'sp_name';
```



### 修改存储过程和函数

**MySQL暂不支持直接修改内容**

```mysql
#修改存储过程或是函数的特性
alter {procedure | function} sp_name [characteristic...]
#sp_name表示存储过程或是函数名称
```



### 删除存储过程和函数

```mysql
drop {procedure | function} [if exists] sp_name
#if exists为MySQL拓展，在没有对应存储过程或是函数的情况下，会抛出一个warning
#通过show warnings可以查看
```

