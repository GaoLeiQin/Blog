**1.定义：**
从一个或多个表中导出来的表，一种虚拟存在的表。

**2.创建视图**

（1）在单表上创建视图

例：

```
create view view_stu as select id ,money from account;

create or replace view view_stu as select id ,money,name from account;

create view view_stu2 (工号,工资,姓名) as select id ,money,name from account;
```

（2）在多表上创建视图

例：

```
create view view_stu3 (NUMMBER,CLASS,NAME) as select account.id,account.name,user.name from account,user where account.id = user.id;
```


**3.查看视图**

例：

（1）查看视图结构：       

```
 desc view_stu;
```


（2）查看视图信息：        

```
show table status like 'stu%';
```


（3）查看视图的创建语句： 

```
 show create view view_stu;
```




**4.修改视图**

例：

```
create or replace view view_stu as select * from account;

alter view dream as select name from possible ;
```



**5.更新视图**

例：

```
update dream set name=Lee where id = 1;

insert into dream values ('LinDan');

delete from dream where name = 'LinDan';
```




**6.删除视图**

例：

```
 drop view dream; 
```

<br/>
<br/>

本人才疏学浅，如有错误，请指出 
谢谢！


