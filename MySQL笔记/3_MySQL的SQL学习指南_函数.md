## 3_MySQL的SQL学习指南_函数



### 数学函数

```mysql
#绝对值
#支持整数和浮点数
select abs(-10),ads(10.5);

#圆周率
#默认显示小数后六位
select pi();

#非负数平方根
#负数返回NULL
select sqrt(40);

#求余函数
#支持整数和浮点数
select mod(10, 2),mod(20.5, 4);


```

