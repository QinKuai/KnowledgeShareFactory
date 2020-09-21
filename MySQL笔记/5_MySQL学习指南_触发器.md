## 5_MySQL学习指南_触发器

触发器与存储过程相似，是**通过事件来触发**的，包括**insert、update和delete**语句。



#### 创建触发器

```mysql
create trigger trigger_name trigger_time trigger_event
on tb_name for each row trigger_stmt
#trigger_name表示触发器名称
#trigger_time表示触发器时机，可以指定before或者after
#trigger_event表示触发器事件，包括insert、update和delete
#trigger_stmt表示触发器程序体

#触发器程序体一行语句可以直接写出，多行需要begin...end括起来
```



#### 查看触发器

```mysql
#直接查看当前的触发器
show triggers;

#通过information_schema.triggers表查看触发器信息
select * from information_schema.triggers;
```



#### 删除触发器

```mysql
drop trigger [dbname.]trigger_name;
```

