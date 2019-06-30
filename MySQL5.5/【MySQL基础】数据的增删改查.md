##1.添加数据

例：先查询表的结构，然后添加数据
![这里写图片描述](http://img.blog.csdn.net/20170417193615117?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
```
insert into student values(1,286.8,'1994-12-28','W');
```
注：
插入数据应与字段的数据类型相同，数据大小应该在范围内。

<br/>
##2.删除数据


（1）删除Lee的信息

```
delete from student where name = 'Lee';
```

（2）删除所有记录（不可以删除表的约束，可以回滚）

```
delete from student ;
```
（3）使用truncate语句删除记录（可以删除表的约束，不可以回滚）

```
truncate student ;
```
<br/>
##3.修改数据

（1）修改所有员工的薪水

```
update employee set salary = 5000;
```

（2）修改Lee的薪水

```
update employee set salary = 9999 where name = 'Lee';
```

（3）给Lee涨1000元工资

```
update employee set salary = salary+1000 where name = 'Lee';
```

<br/>
##4.查询数据


**（1）简单查询**

例1：查询表中所有学生的信息

```
select * from student;
```

例2：查询表中所有学生姓名和对应的英语成绩

```
select name,English from student;
```
例3：过滤表中重复的数据

```
select distinct English from student;
```
例4：统计每个学生总分

```
select name,math+English+chinese from student;
```
例5：使用别名表示

```
select name 姓名,math+English+chinese 总成绩 from student;
```
<br/>
**（2）使用where语句查询**

例1：查询Lee的成绩

```
select * from student where name = 'Lee';
```
例2：查询英语成绩大于60分的学生

```
select * from student where English > 60;
```
例3：查询英语成绩在80~100分的学生

```
select * from student where English between 80 and 100;
```
例4：查询数学成绩是66，77，88，99的学生

```
select * from student where math in(66，77，88，99);
```
例5：查询所有姓张的学生成绩

```
select * from student where name like '张%';
```
注:
名字是两个字 '张_%'
名字是三个字 '张_ _%'

例6：查询数学成绩>80分 且 英语成绩>60分的学生

```
select * from student where math > 80 and English > 60;
```

例7：查询地址有误的学生（即null和空串）

```
select * from student where address is null or address = '';
```

<br/>
**（3）排序查询（默认ASC升序 ，  desc降序）**

例1：对英语成绩排序后输出

```
select name,English from exam order by English; 
```
例2：对姓张的学生总成绩由高到低输出

```
select name 姓名,math+English+chinese 总分 from student where name like '张%' order by 总分 desc;
```
<br/>
**（4）利用聚合函数查询**

*count（） 函数 ： 统计 行数 某列

例1：统计一个班学生总数

```
select count(*) from student;
```

例2：统计数学成绩>90分的学生人数

```
select count(*) from where math > 90;
```

*math（）函数： 统计某列值的和

例1：统计一个班数学总成绩

```
select sum(math) from student;
```
例2：统计一个班各科总成绩

```
select sum(math),sum(English) from student;
```

*avg（）函数：求某列平均值

例1：求某班数学平均分

```
select avg(math) from student;
```

例2：求班级总平均分

```
select avg(math+English+chinese) from student;
```

*max（）函数求某列最大值

例：求最高分

```
select max(math+English+chinese) from student;
```

*min（）函数求某列最小值

例：求最低分

```
select min(math+English+chinese) from student;
```

**（5）分组查询**

例1：对订单表的商品归类，显示每一类商品的总价

```
select product, sum(price) from orders group by product;
```

例2：查询总价>100元商品的名称

```
select product, sum(price) from orders group by product having sum(price) > 100;
```
例3：查询单价<100元，总价>150的商品的名称

```
select product, sum(price) from orders group by product < 100 having sum(price) > 150;
```
例4：使用limit限制查询结果

```
select * from orders limit 5,3;
```
注：不包括第5行（即第6，7，8行数据）

<br/>

**（6）其它函数**

*abs（）   绝对值
*length（）  返回字符串长度
*curdate（） 获取当前日期

<br/>
##5.date的用法

（1）显示当前的日期时间

例1：显示日期时间
```
select now() from dual;
```
例2:只显示日期

```
select current_date() from dual;
```

（2）给当前的时间加20分钟

```
select date_add(now(),interval 20 minute) from dual;
```

（3）查询两小时内发布的消息

```
select * from message where date_add(today,interval 2 hour) >= now();
```

（4）查询两个日期相差几天

```
datediff (now(),hiredate) as decreasedate;
```

<br/>
<br/>
本人才疏学浅，如有错误，请指出 
谢谢！


