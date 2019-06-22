**1.创建表**

例：创建一张名为employee的表
![这里写图片描述](http://img.blog.csdn.net/20170417171013697?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.查看表结构**

（1）显示表的结构

例：查看employee的表结构

![这里写图片描述](http://img.blog.csdn.net/20170417173309450?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）查看表的建表语句

例：查看表employee的建表语句

![这里写图片描述](http://img.blog.csdn.net/20170417173529905?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）查看数据库中的所有表

例：查看数据库dream中的所有表

![这里写图片描述](http://img.blog.csdn.net/20170417173819283?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
**3.修改表的结构**

（1）增加列

例：给表employee增加一列，表名为image，数据类型为blob

![这里写图片描述](http://img.blog.csdn.net/20170417174222602?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）修改列

例：修改表employee中salary列的数据类型为float,并把salary列置顶

![这里写图片描述](http://img.blog.csdn.net/20170417174721864?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）删除列

例：删除表employee中的gender列

![这里写图片描述](http://img.blog.csdn.net/20170417174947924?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（4）修改字符集

例：将表employee的字符集修改为GBK

![这里写图片描述](http://img.blog.csdn.net/20170417175148382?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（5）修改列名

例：将表employee中列name修改为username

![这里写图片描述](http://img.blog.csdn.net/20170417175556823?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（6）更改表名

例：将表employee更改为 user

方式一：

![这里写图片描述](http://img.blog.csdn.net/20170417180023247?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

方式二：

![这里写图片描述](http://img.blog.csdn.net/20170417180147893?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（7）删除表

例：删除表student

![这里写图片描述](http://img.blog.csdn.net/20170417180338842?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br/>

**4.表的约束**

（1）主键约束

*单字段

```MySQL
id int primary key,
```
*多字段

```MySQL
primary key (id int, name varchar(20)),
```

（2）非空约束

```MySQL
id int not null,
```

（3）唯一约束

```MySQL
name varchar(20) unique,
```
（4）默认约束

```MySQL
salary float default 1000,
```

（5）字段值自动增加

```MySQL
id int auto_increment,
```
<br/>
**5.创建索引**

（1）创建新索引（创建表时创建索引）

*创建普通索引
```MySQL
index (id),
```
*创建唯一索引
```MySQL
unique index unique_id (id asc),
```

*创建全文索引
```MySQL
fulltext index fulltext_name (name),
```
注：engine=MyISAM

*创建单列索引
```MySQL
index single_name (name(20)),
```

*创建多列索引
```MySQL
index multi(id, name(20)),
```

*创建空间索引
```MySQL
spatial index sp (space),
```
<br/>
（2）使用create index语句在已经存在的表上创建索引

*创建普通索引

```
create index index_id on employee (id);
```

*创建唯一索引

```
create unique index unique_id on employee (id);
```
*创建全文索引

```
create fulltext index fulltext_name on employee (name);
```

*创建单列索引

```
create index single_id on employee (id);
```
*创建多列索引

```
create index multi on employee (id,name(20));
```
*创建空间索引

```
create spatial index spatidx on employee (space);
```
<br/>
（3）使用alter table 语句在已存在的表上创建索引

*创建普通索引

```
alter table employee add index idx(id);
```

*创建唯一索引

```
alter table employee add unique uniqueidx(id);
```
*创建全文索引

```
alter table employee add fulltext index fulltextidx(id);
```
*创建单列索引

```
alter table employee add single singleidx(id);
```
*创建多列索引

```
alter table employee add multi multidx(id,name(20));
```
*创建空间索引

```
alter table employee add spatial spatidx(space);
```

**6.删除索引**

（1）使用alter table 语句删除索引

```
alter table employee drop index fulltextidx;
```

（2）使用index删除索引

```
drop index spatidx on employee;
```
<br/>
<br/>
本人才疏学浅，如有错误，请指出 
谢谢！