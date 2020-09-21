## 2_MySQL的SQL学习指南_数据类型与运算符



#### 数据类型

##### 整数类型

`无符号数在数据类型后加上unsigned即可`

| 名称         | 存储要求(Byte) | 有符号范围               | unsigned   |
| ------------ | -------------- | ------------------------ | ---------- |
| tinyint      | 1              | -128~127                 | 0~255      |
| smallint     | 2              | -32768~32767             | 0~65535    |
| mediumint    | 3              | -838万~838万             | 0~1677万   |
| int(integer) | 4              | -21.47亿~21.47亿         | 0~42.94亿  |
| bigint       | 8              | -9.22\*10^19~9.22\*10^19 | 1.8\*10^20 |





##### 浮点数

`浮点数的取值范围非常的大，一般不用考虑范围问题，但要考虑精度问题，默认精度与操作系统相关，最好不要用浮点数做计算和比较`

| 名称   | 存储要求(Byte) |
| ------ | -------------- |
| float  | 4              |
| double | 8              |



##### 日期和时间

| 名称      | 日期格式            | 日期范围                                         | 存储要求(Byte) |
| --------- | ------------------- | ------------------------------------------------ | -------------- |
| year      | yyyy                | 1901-2155                                        | 1              |
| time      | hh:mm:ss​            | -838:59:59~838:59:59                             | 3              |
| date      | yyyy-mm-dd          | 1000-01-01~9999-12-31                            | 3              |
| datetime  | yyyy-mm-dd hh:mm:ss | 1000-01-01 00:00:00~9999-12-31 00:00:00          | 8              |
| timestamp | yyyy-mm-dd hh:mm:ss | 1970-01-01 00:00:01 UTC ~2038-01-19 03:14:07 UTC | 4              |

```mysql
#year
#year可以通过字符串或数字输入

#time
#time不仅可以表达当天的时间，还能表达时间差
#系统日期函数做默认值
#type 1
create table tb_1(
login_time time default (current_time)
);
#type 2
create table tb_1(
login_time time default (current_time())
);
#type 3
create table tb_1(
login_time time default (now())
);
#type 4
create table tb_1(
login_time time default (curtime())
);

#date
#默认值插入系统时间
#type 1
create table tb_1(
login_time date default (current_date())
)
#type 2
create table tb_1(
login_time date default (now())
)
#type 3
create table tb_1(
login_time date default (curdate())
)

#datetime
#默认值设置为系统时间
#type 1
create table tb_1(
login_time datetime default (now())
);

#timestamp

```

