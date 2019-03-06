####**1.MySQL常用命令**

* 创建表：create table employee(id int,name varchar(20));
* 修改表结构（列）：alter table employee add/modify/drop salary float;
 * 修改字符集：alter table employee character set GBK;
 * 修改列名：alter table employee change math English int;
 * 更改表名：alter table employee rename student;
* 删除表：drop table student;
* 创建索引：create unique index unique_id on employee (id);或alter table employee add fulltext index fulltextidx(id);
* 删除索引：drop index unique_id on employee;或alter table employee drop index fulltextidx;
* 添加数据：insert into student values(1,"W");
* 删除数据：delete from student where name = 'Lee';
* 修改数据：update employee set salary = 5000;
* 查询数据：
 * select distinct English from student;
 * select * from student where English between 80 and 100;
 * select * from student where name like '张%';
 * select name 姓名,math 数学from student where name like '张%' order by 数学 desc;
 * select count(*)/sum(math)/avg(math) from student;
 * select product, sum(price) from orders group by product having sum(price) > 100;
* 查询日期：select now() from dual;或select date_add(now(),interval 20 minute) from dual;
* 添加外键约束：alter table student add constraint FK_ID foreign key (gid) references grade (id) ;（先添加主表，先修改或删除副表）
* 左外连接查询：select * from dept left join possible on dept.id = possible.dept_id;
* 子查询：select *from department where did>any(select did from employee);
* 备份数据库：mysqldump -u username -p mydb2 >c:/311.sql
* 创建触发器：create trigger newproduct after insert on products for each now select "product added";

####**2.数据库中事物的特征？**

* 事务：start transaction;、commit; 、rollback; 

* 事务的四大特性
 * 原子性：事务是一组不可分割的单位，要么同时成功要么同时不成功
  * 一致性：事务前后的数据完整性应该保持一致， 
  * 隔离性：多个用户并发访问数据库时，一个用户的事务不能被其他用户的事务所干扰 
  * 持久性：一个事务一旦被提交，对数据库中的数据的改变就是永久性的，即使数据库发生故障也不应该对其有任何影响。

* 读的种类
 * 脏读：一个事务读取到另一个事务未提交的数据
 * 不可重复读：一个事务多次读取同一条记录，读取的结果不相同
 * 虚读（幻读）：一个事务多次查询整表的数据，由于其他事务新增（删除）记录造成多次查询出的记录条数不同。

* 四大隔离级别

|隔离级别	|脏读（Dirty Read）	|不可重复读（NonRepeatable Read）|	幻读（Phantom Read）|
|:-----|:-----|:------|:-----|
|未授权读（Read uncommitted）|	可能|	可能	|可能|
|已授权读（Read committed）|	不可能	|可能|	可能|
|可重复读（Repeatable read）|	不可能	|不可能	|可能|
|可串行化（SERIALIZABLE）|	不可能	|不可能|	不可能|

注：mysql 默认是repeatable read 级别！

####**3.JDBC的使用？**

* 使用步骤
 * 注册驱动：Class.forName("com.mysql.jdbc.Driver");
 * 建立连接：Connection conn = DriverManager.getConnection(URL, user, password); 
 * 执行SQL语句：PreparedStatement ps = conn.prepareStatement(sql);
 * PreparedStatement 可用预编译的sql、sql缓存区、有效防止sql注入（OR 1=1 ）
 * 执行处理结果：ResultSet rs = ps.executeQuery();

* 常用组件
 * BeanUtils：对象的拷贝、注册日期类型转换器
 * DbUtils：定义了所有的与数据库操作的方法，简化编码量
 * C3P0连接池：new ComboPooledDataSource();


####**4.InnodB与MyISAM的区别**

* 建立索引的目的是加快对表中记录的查找或排序，但索引是占用空间的，影响速度。索引类型：主键索引、唯一索引、普通索引、全文索引（FULLTEXT ），而主键索引是特殊的唯一性索引。

* MySQL支持MyISAM、InnoDB、MEMORY、MERGE、ARCHIVE多种存储引擎，而默认是InnoDB

* MyISAM：独立于操作系统，表锁，不支持外键、事务。存储格式分为静态、动态、压缩表，适合以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性，并发性要求不是很高的情况

* InnoDB：默认，行级锁定、外键约束、自动灾难恢复、支持事务。对事物的完整性有比较高的要求，在并发条件下要求数据的一致性，包含很多操作。InnoDB只有通过索引进行检索的时候才会使用行级锁，否则会使用表级锁。

* MEMORY：存在内存，为得到最快的响应时间；MERGE：一组MyISAM表的组合；ARCHIVE：归档后仅支持插入和查询。

* 实现方式：
 * MyISAM：使用B+ Tree作为索引结构，叶节点存放的是数据记录的地址，索引和实际的数据是分开的，只不过是用索引指向了实际的数据，即非聚集索引。
 * InnoDB：数据本身就是按B+ Tree组织的一个索引结构，叶节点存放的是完整的数据记录，即聚集索引，主键是InnoDB表记录的”逻辑地址“，所以InnoDB要求表必须有主键，MyISAM可以没有

####**5.MySQL为什么使用B+树作为索引？**

* 数据结构作为索引的优劣指标：索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数。

* B树适合在磁盘等直接存储设备上组织动态查找表，文件的组织方式是B树或B+树。在查询和动态操作方面效率很高。虽然hash表的效率更高，可是hash表存在散列冲突问题，而且一般利用率仅为50%，当存储比例达到一定程度时hash表必须进行扩容才能维持之后的操作，而B树不用。


<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！