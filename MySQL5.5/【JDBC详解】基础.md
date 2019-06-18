
##一、概念

**1.定义**

JDBC (Java Database Connectivity)，是一组标准的Java语言中的接口和类，它的意义是在Java程序中执行SQL语句。

**2.数据库驱动**

应用程序、JDBC API、数据库驱动及数据库之间的关系

![应用程序、JDBC API、数据库驱动及数据库之间的关系](http://img.blog.csdn.net/20170429104038494?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
**3.常用的类和接口**

（1）Driver接口 ： 表示java驱动程序接口。

常用方法：

* connect(url, properties):  连接数据库的方法。
* url: 连接数据库的URL 
* url语法： jdbc协议:数据库子协议://主机:端口/数据库
* user： 数据库的用户名
* password： 数据库用户密码

（2）DriverManager类 ： 驱动管理器类，用于管理所有注册的驱动程序

常用方法

* registerDriver(driver)  : 注册驱动类对象
* Connection getConnection(url,user,password);  获取连接对象

（3）Connection接口： 表示java程序和数据库的连接对象。

常用方法

* Statement createStatement() ： 创建Statement对象来将 SQL 语句发送到数据库
* PreparedStatement prepareStatement(String sql)  创建PreparedStatement对象来将参数化的 SQL 语句发送到数据库
* CallableStatement prepareCall(String sql) 创建CallableStatement对象

（4）Statement接口： 用于执行静态的sql语句

常用方法

* int executeUpdate(String sql)  ： 执行静态的更新sql语句（DDL，DML）
* ResultSet executeQuery(String sql)  ：执行的静态的查询sql语句（DQL）

（5）PreparedStatement接口：用于执行预编译sql语句

常用方法

* int executeUpdate() ： 执行预编译的更新sql语句（DDL，DML）
* ResultSet executeQuery()  ： 执行预编译的查询sql语句（DQL）

（6）CallableStatement接口：用于执行存储过程的sql语句（call xxx）

常用方法

* ResultSet executeQuery()  ： 调用存储过程的方法

（7）ResultSet接口：用于封装查询出来的数据

常用方法

*  boolean next() ： 将光标移动到下一行
* getXX() : 获取列的值

图解：

![这里写图片描述](http://img.blog.csdn.net/20170429110842778?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
##二、使用JDBC的步骤

主要步骤： 加载JDBC驱动程序 → 建立数据库连接 → 创建执行SQL的语句 → 处理执行结果 → 释放资源

**1.注册驱动**

方式一：通过得到字节码对象的方式加载静态代码块，从而注册驱动程序（推荐使用）

```
Class.forName("com.mysql.jdbc.Driver");
```

方式二：使用驱动管理器类连接数据库，驱动注册了两次（不常用）

```
Driver driver = new com.mysql.jdbc.Driver();
DriverManager.registerDriver(driver)；

```
**2.建立连接**

```
Connection conn = DriverManager.getConnection(URL, user, password); 
```

URL用于标识数据库的位置，通过URL地址告诉JDBC程序连接哪个数据库，URL的写法为：

![这里写图片描述](http://img.blog.csdn.net/20170429110824590?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.创建执行SQL语句**


（1）使用PreparedStatement执行sql语句（推荐使用，可以防止SQL注入）

```
String sql = "SELECT * FROM student"; 
PreparedStatement stmt = conn.prepareStatement(sql);
stmt.executeQuery();
```

（2）使用Statement执行SQL语句
```
Statement stmt = conn.createStatement();
String sql = "INSERT INTO student(NAME,gender) VALUES('林丹','男')";
stmt.executeUpdate(sql);
```


（3） PreparedStatement 与 Statment的区别

* 语法不同：PreparedStatement可以使用预编译的sql，而Statment只能使用静态的sql
* 效率不同： PreparedStatement可以使用sql缓存区，效率比Statment高
* 安全性不同： PreparedStatement可以有效防止sql注入，而Statment不能防止sql注入。


**4.处理执行结果**

```
ResultSet rs = ps.executeQuery();
while(rs.next()){
	int id = rs.getInt("id");
	String name = rs.getString("name");
	String gender = rs.getString("gender");
	System.out.println(id+","+name+","+gender);
}
```
**5.释放资源**

```
try {
     if (rs != null) {
         rs.close();
     }
 } catch (SQLException e) {
    e.printStackTrace();
 }
 
try {
     if (stmt != null) {
         stmt .close();
      }
} catch (SQLException e) {
    e.printStackTrace();
}

try {
     if (conn != null) {
         conn.close();
      }
} catch (SQLException e) {
     e.printStackTrace();
}
```

**6.完整的例子**

例：建立连接执行查询数据库

```
public class CreateMysql {
    public static void main(String[] args) {
        Connection ct = null;
        PreparedStatement ps = null;
        ResultSet rs = null;

        try {
            //通过得到字节码对象的方式加载静态代码块，从而注册驱动程序
            Class.forName("com.mysql.jdbc.Driver");
            //得到连接
            ct = DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","965847");
            //创建ps
            ps = ct.prepareStatement("SELECT * FROM students");
            //执行
            rs = ps.executeQuery();
            while (rs.next()) {
                System.out.println("学号： "+rs.getInt("id")+" 科目："+rs.getString("subject")+"  成绩： "+rs.getFloat("score"));
            }

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (rs != null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
           
            if (ps != null){
                try {
                    ps.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            
            if (ct != null){
                try {
                    ct.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

##三、执行存储过程

使用CablleStatement调用存储过程

**1.调用带有输入参数的存储过程**

（1）MySQL的存储过程（有输入参数）如下

![这里写图片描述](http://img.blog.csdn.net/20170429114201840?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）调用存储过程

例：

```
public class CallInProcedure {
    public static void main(String[] args) {
        Connection ct = null;
        PreparedStatement ps = null;
        ResultSet rs = null;

        try {
            Class.forName("com.mysql.jdbc.Driver");
            ct = DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","965847");
            String sql = "CALL pro_findId(?)";
            //预编译
			ps = conn.prepareCall(sql);
			//设置输入参数
			ps.setInt(1, 3);
            //发送参数（所有调用存储过程的sql语句都是使用executeQuery方法执行）
            rs = ps.executeQuery();
            while(rs.next()){
				int id = rs.getInt("id");
				String username = rs.getString("usernName");
				String password = rs.getString("pwd");
				System.out.println(id+","+username +","+password );
			}
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (rs != null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

            if (ps != null){
                try {
                    ps.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

            if (ct != null){
                try {
                    ct.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

<br/>

**2.调用带有输出参数的存储过程**

（1）MySQL的存储过程（有输出参数）如下

注：@pwd 为用户变量
![这里写图片描述](http://img.blog.csdn.net/20170429120547084?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）调用存储过程

例：

```
public class CallOutProcedure {
    public static void main(String[] args) {
        Connection ct = null;
        PreparedStatement stmt = null;

        try {
            Class.forName("com.mysql.jdbc.Driver");
            ct = DriverManager.getConnection("jdbc:mysql://localhost:3306/test","root","965847");
            //第一个？是输入参数，第二个？是输出参数
            String sql = "CALL findpwd(?,?)"; 
			//预编译
			stmt = conn.prepareCall(sql);
			//设置输入参数
			stmt.setInt(1, 3);
			//设置输出参数(注册输出参数)
			stmt.registerOutParameter(2, java.sql.Types.VARCHAR);
			//发送参数(结果不是返回到结果集中，而是返回到输出参数中)
			stmt.executeQuery();
			//得到输出参数的值（getXX方法专门用于获取存储过程中的输出参数）
			String result = stmt.getString(2);
			System.out.println(result);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (stmt != null){
                try {
                    stmt .close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

            if (ct != null){
                try {
                    ct.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

<br/>
<br/>
本人才疏学浅，若有错误，请指出
谢谢！