**1.数据的备份**

（1）单个数据库备份

```
mysqldump -u username -p mydb2 >c:/311.sql
```

（2）备份所有数据库

```
mysqldump -u username -p --all-databases >c:/515.sql
```


**2.数据的还原**

（1）恢复数据库

```
 mysql -u root -p mydb3 < 311.sql
```


(2) source命令

*创建数据库  

```
create database mydb3;
```

*进入数据库  

```
use mydb3;
```

*输入命令       

```
 source c:/311.sql;
```


**3.用户管理**


(1)创建普通用户

例：

```
grant select on mydb3.* to 'ABCDEFG'@'localhost' identified by '123';

create user 'huochewang'@'localhost' identified by '311515';
    frush privileges;

insert into mysql.user(Host,user,Password,ssl_cipher,x509_issuer,x509_subject)
   values ('localhost','huochewang',password('515666'),'','','');

frush privileges;
```



(2)删除用户

```
drop user 'huochewang3'@'localhost';

delete from mysql.user where host='localhost' and user='huochewang2';

frush privilages;
```


(3)修改用户密码

*修改root用户密码

*     mysqladmin -u root -p password 654321

*    set password=password('123456')；

*   update mysql.user set password=password('abc') where user='root' and host='localhost';

*   frush privilages;


*利用root用户修改普通用户的密码

*   mysqladmin -u root -p password it123

*   set password for 'username'@'hostname'=password<'dreamit'>;

*   update mysql.user set password=password('abc') where user='root' and host='localhost';

*   frush privilages;




*普通用户修改自己的密码

*  set password = password('666311515');


**4.权限管理**


(1)授予权限

grant option; 将自己的权限授予其他用户
例：

```
grant insert,select on *.* to 'user4'@'localhost' identified by '123'with grant option;
```



(2)查看权限

```
show grants for 'root'@'localhost';
```

注：@前后不能有空格。

(3)收回权限

*收回insert权限

```
 revoke insert on *.* from 'user4'@'localhost';
```


*收回所有权限

```
revoke all privileges,grant option from 'user4'@'localhost';
```

<br/>
<br/>
本人才疏学浅，如有错误，请指出 
谢谢！