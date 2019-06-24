**1.外键**

（1）定义：
引用另一个表中的一列或多列，被引用的列应该具有主键约束或唯一性约束，外键用于建立和加强两个表数据之间的连接。

（2）为表添加外键约束

```
alter table student add constraint FK_ID foreign key (gid) references grade (id) ;
```
注： gid外键依赖于grade表中的id主键，gid是student的外键。

（3）删除外键约束

```
alter table student drop foreign key FK_ID;
```

例：

部门表（主表）
```
CREATE TABLE dept(
	id INT PRIMARY KEY,
	deptName VARCHAR(20)
)
```

修改员工表（副表/从表）

```
CREATE TABLE employee(
	id INT PRIMARY KEY,
	empName VARCHAR(20),
	deptId INT,
	CONSTRAINT emlyee_dept_fk FOREIGN KEY(deptId) REFERENCES dept(id)
	--           外键名称                  外键               参考表(参考字段)
)
```
 注意：

* 被约束的表称为副表，约束别人的表称为主表，外键设置在副表上的！！！
* 主表的参考字段通用为主键！
* 添加数据： 先添加主表，再添加副表
* 修改数据： 先修改副表，再修改主表
* 删除数据： 先删除副表，再删除主表

（4）级联操作
出现原因：当有了外键约束的时候，必须先修改或删除副表中的所有关联数据，才能修改或删除主表！我们希望直接修改或删除主表数据，从而影响副表数据。就可以使用级联操作实现！

级联修改： ON UPDATE CASCADE
级联删除： ON DELETE CASCADE

例：

```
CONSTRAINT emlyee_dept_fk FOREIGN KEY(deptId) REFERENCES dept(id) ON UPDATE CASCADE ON DELETE CASCADE
```

应用：

* 级联修改（修改）

例：直接修改部门

```
UPDATE dept SET id=5 WHERE id=4;
```

* 级联删除

例：直接删除部门 

```
DELETE FROM dept WHERE id=1;

```

<br/>
**2.操作关联表**

（1）关联关系

*多对一  例：员工与部门之间的关系
*多对多  例：学生与课程之间的关系
*一对一  例：公民与身份证之间的关系

（2）添加数据

*为主表grand添加数据

```
insert into grade (id,name) values(1,'oneClass');
insert into grade (id,name) values(2,'twoClass');
```
注：由于student表的外键与grade表的主键关联，因此在为student表添加数据时，gid的值只能是1或2。

（3）删除数据

```
delete from grade where id = 1;
```
<br/>
**3.连接查询**

（1）笛卡尔查询：两张表相乘的结果

```
select * from dept,possible;
```
（2）内连接查询：查询出左边表右边表都有的数据记录。

```
select * from dept,possible where dept.id = possible.dept_id;

select * from dept inner join possible on dept.id = possible.dept_id;
```

（3）外连接查询

*左外连接查询，在内连接的基础上增加左边表有而右边表没有的记录
```
select * from dept left join possible on dept.id = possible.dept_id;
```

*右外连接查询，在内连接的基础上增加右边表有而左边表没有的记录

```
select * from dept right join possible on dept.id = possible.dept_id;
```
*复合条件连接查询

```
select *from dept full join possible on dept.id = possible.dept_id;

```

注：mysql不支持全外连接查询，mysql中合并左外右外查询从而达到全外连接查询的目的

<br/>
**4.子查询**

（1）带IN关键字的子查询

```
select dname from departmaent where did in (select did from employee where age=20);
```

注：not in 关键字与in作用相反；

（2）带EXISTS关键字的子查询（不产生任何数据，只返回TRUE或FALSE）

```
select * from department where exists(select * from employee where age>21);
```
（3）带ANY关键字的子查询

```
select *from department where did>any(select did from employee);
```

（4）带ALL关键字的子查询

```
select * from department where did >all (select did from employee);
```

（5）带比较运算符的子查询

```
select dname from department where id = (select did from employee where name='Lee');
```

<br/>
<br/>
本人才疏学浅，如有错误，请指出 
谢谢！