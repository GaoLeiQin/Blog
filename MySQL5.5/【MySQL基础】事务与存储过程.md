**1.事务的概念**

（1）事务：逻辑上的一组操作，MySQL默认自带事务，但是一个语句独占一个事务，也可以自己来控制事务。

 * start transaction ---开启事务，在这条语句之后的sql将处于同一事务中，并不会立即影响到数据库
 * commit --提交事务，让这个事务中的sql对数据库的影响立即发生
 * rollback -- 回浪事务，取消这个事务，这个事务不会对数据库中的数据产生影响了

**2.事务的四大特性**

（1）原子性：事务是一组不可分割的单位，要么同时成功要么同时不成功

（2）一致性：事务前后的数据完整性应该保持一致，
（数据库的完整性：如果数据库在某个时间点下，所有的数据都符合所有的约束）

（3）隔离性：多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰
多个并发事务之间数据要相互隔离。

（4）持久性：一个事务一旦被提交，他对数据库中的数据的改变就是永久性的
即使数据库发生故障也不应该对其有任何影响。

**3.读与隔离级别**

（1）读

*脏读：一个事务读取到另一个事务未提交的数据。

*不可重复读：一个事务多次读取同一条记录，读取的结果不相同
（一个事务读取到另一个事务已经提交的数据）

*虚读（幻读）：一个事务多次查询整表的数据，由于其他事务新增（删除）记录造成多次查询出的记录条数不同。

（2）四大隔离级别

|隔离级别	|脏读（Dirty Read）	|不可重复读（NonRepeatable Read）|	幻读（Phantom Read）|
|:-----|:-----|:------|:-----|
|未授权读（Read uncommitted）|	可能|	可能	|可能|
|已授权读（Read committed）|	不可能	|可能|	可能|
|可重复读（Repeatable read）|	不可能	|不可能	|可能|
|可串行化（SERIALIZABLE）|	不可能	|不可能|	不可能|

注：mysql 默认是repeatable read 级别！

**4.流程控制语句的使用**

（1）IF 语句

例：

```
delimiter //
 create procedure Pro3()
    begin
    declare a varchar(20);
    if a is null then select 'is null';
    else select 'is not null';
    end if; 
END //

delimiter ;
```

（2）CASE 语句

例：

```
delimiter //
 create procedure Pro4()
    begin
    declare b int;
    case b
        when 1 then select 'value is 1';
        when 2 then select 'value is 2';
    end case; 
END //

delimiter ;
```

(3)LOOP 语句
例：

```
create procedure Pro5()
    begin
    declare id int default 0;
    my_loop:Loop 
        set id = id+1;
        if id<10 then iterate my_loop;
        elseif id>20 then leave my_loop;
        end if;
        select 'id is between 10 and 20';
    end Loop my_loop; 
END

```

注：leave 跳出循环
    iterate 再次循环

(4)REPEAT 语句

例： 

```
  declare id int default 0;
    repeat
        set id = id+1;
        untill id>=10 ;
    end Loop REPEAT; 
```

（5）WHILE语句
例：

```
 declare idcard int default 0;
   while i<10 Do
        set idcard = idcard+1;
   end while;

```

**5.存储过程的使用**

(1)执行存储过程
例：

```
delimiter //
 create procedure Pro3()
    begin
    declare a varchar(20);
    if a is null then select 'is null';
    else select 'is not null';
    end if; 
END //

call Pro3();

```


(2)查看存储过程

```
* show procedure status;
* show procedure status like '%2';

* show create procedure Proc4;

* select * from information_schema.Routines;

```


(3)修该存储过程(特性)

```
alter produce Pro3 modifies sql data sql security invoker;
```
(4)删除存储过程
例：

```
drop procedure Proc4;
```

**6.光标的使用**

（1）光标的声明

```
declare cursor_student cursor for select s_name from student;
```

（2）光标的使用

*使用名称为cursor_student的光标，将查询出的信息存入s_name

```
fetch cursor_student into s_name;
```

（3）光标的关闭

```
close cursor_student;
```

（4）打开光标

```
open cursor_student;
```

<br/>
<br/>
本人才疏学浅，如有错误，请指出 
谢谢！