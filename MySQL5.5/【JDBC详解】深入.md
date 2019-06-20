###1.SQL注入攻击

（1）定义：

SQL 注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的 SQL 语句段或命令，从而利用系统的 SQL 引擎完成恶意行为的做法。

（2）解决方法：用 PreparedStatement（预编译） 取代 Statement 就可以了

原因：使用预编译SQL语句避免了频繁sql拼接 (可以使用占位符)，也可以防止sql注入

例：OR 1=1 语句将删除整个表 ;

![这里写图片描述](http://img.blog.csdn.net/20170429133429184?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
<br/>
###2.批处理

（1）需求：批量保存信息。

（2）常用方法

* void addBatch(String sql)     添加批处理
* void clearBatch()            清空批处理
* int[] executeBatch()         执行批处理

（3）进行批处理

例：

```
public void batching(List<Admin> list) {

        private Connection con = null;
	    private PreparedStatement pstmt = null;
		String sql = "INSERT INTO admin(userName,pwd) values(?,?)";
		
		try {
			
			// 获取连接（利用工具类）
			con = JdbcUtil.getConnection();
			pstmt = con.prepareStatement(sql);   //预编译SQL语句
			
			for (int i=0; i<list.size(); i++) {
				Admin admin = list.get(i);
				// 设置参数
				pstmt.setString(1, admin.getUserName());
				pstmt.setString(2, admin.getPwd());
				// 添加批处理
				pstmt.addBatch();				// 不需要传入SQL
				// 测试：每5条执行一次批处理
				if (i % 5 == 0) {
					// 批量执行 
					pstmt.executeBatch();
					// 清空批处理
					pstmt.clearBatch();
				}
				
			}
			
			// 批量执行 
			pstmt.executeBatch();
			// 清空批处理
			pstmt.clearBatch();
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
		    //（利用工具类）
			JdbcUtil.closeAll(con, pstmt, null);
		}
	}
}

```

###3.	事务

(1)概念：

事务使指一组最小逻辑操作单元，里面有多个操作组成。 组成事务的每一部分必须要同时提交成功，如果有一个操作失败，整个操作就回滚。

（2）四个特性（ACID）

	原子性（Atomicity）
原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。 

	一致性（Consistency）
事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

	隔离性（Isolation）
事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。

	持久性（Durability）
持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

（3）特性的总结：

＊　原子性，是一个最小逻辑操作单元 !
＊　一致性，事务过程中，数据处于一致状态。
＊　持久性， 事务一旦提交成功，对数据的更改会反映到数据库中。
＊　隔离性， 事务与事务之间是隔离的。

（４）JDBC事务处理


* 当一个连接对象被创建时，默认情况下是自动提交事务：每次执行一个 SQL 语句时，如果执行成功，就会向数据库自动提交，而不能回滚

* 为了让多个 SQL 语句作为一个事务执行，调用 Connection 对象的 setAutoCommit(false); 以取消自动提交事务

* 在所有的 SQL 语句都成功执行后，调用 commit(); 方法提交事务

* 若出现异常，调用 rollback(); 方法回滚事务

* 若此时 Connection 没有被关闭, 则需要恢复其自动提交状态

例：

```
public void trans() {

		String sql_zs = "UPDATE account SET money=money-1000 WHERE accountName='Lee';";
		String sql_ls = "UPDATE1 account SET money=money+1000 WHERE accountName='LinDan';";

		try {
			con = JdbcUtil.getConnection();
			//设置手动提交任务
			con.setAutoCommit(false);

			pstmt = con.prepareStatement(sql_zs);
			pstmt.executeUpdate();

			pstmt = con.prepareStatement(sql_ls);
			pstmt.executeUpdate();

		} catch (Exception e) {
			try {
				//若出现异常则回滚
				con.rollback();
			} catch (SQLException e1) {
			}
			e.printStackTrace();
		} finally {
			try {
				//若无异常则提交事务
				con.commit();
				//工具类
				JdbcUtil.closeAll(con, pstmt, null);
			} catch (SQLException e) {
			}
		}

	}
```

###4.大文本类型的处理

（1）MySQL的大文本类型，

* Text    长文本类型
* Blob    二进制类型

例:

```
public void testSaveText() {
		String sql = "insert into test(img) values(?)";
		try {
			// 连接
			con = JdbcUtil.getConnection();
			// pstmt 对象
			pstmt = con.prepareStatement(sql);
			// 获取图片流
			InputStream in = App_text.class.getResourceAsStream("background.jpg");
			pstmt.setBinaryStream(1, in);
			
			// 执行保存图片
			pstmt.execute();
			
			// 关闭
			in.close();
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			JdbcUtil.closeAll(con, pstmt, null);
		}
	}

```

<br/>
<br/>

本人才疏学浅，若有错误，请指出
谢谢！