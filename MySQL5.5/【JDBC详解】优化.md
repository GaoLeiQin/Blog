**1.BeanUtils组件**

（1）出现原因：

程序中对javabean的操作很频繁， 所以apache提供了一套开源的api，方便对javabean的操作！即BeanUtils组件。

（2）作用：

简化javabean的操作！

（3）下载：

从www.apache.org下载BeanUtils组件，然后再在项目中引入jar文件！

（4）常用方法

* 对象属性的拷贝

	BeanUtils.copyProperty(admin, "userName", "jack");
BeanUtils.setProperty(admin, "age", 18);

* 对象的拷贝

	BeanUtils.copyProperties(newAdmin, admin);

* map数据拷贝到javabean中  （map中的key要与javabean的属性名称一致）

	BeanUtils.populate(adminMap, map);

* 注册日期类型转换器（使用组件提供的转换器工具类）
	
	ConvertUtils.register(new DateLocaleConverter(), Date.class);


例：

```
public void test1() throws Exception {
		
		// 1. 基本操作
		Admin admin = new Admin();
		admin.setUserName("Jack");
		admin.setPwd("999");
		String birth ="2017-04-29";
		
		// 2. BeanUtils组件实现对象属性的拷贝
		BeanUtils.copyProperty(admin, "userName", "jack");
		BeanUtils.setProperty(admin, "age", 18);
		//对于基本数据类型，会自动进行类型转换
		
		// 3. 对象的拷贝
		Admin newAdmin = new Admin();
		BeanUtils.copyProperties(newAdmin, admin);
		
		// 4. map数据，拷贝到对象中
		Admin adminMap = new Admin();
		Map<String,Object> map = new HashMap<String,Object>();
		map.put("userName", "Jerry");
		map.put("age", 29);
		// 注意：map中的key要与javabean的属性名称一致
		BeanUtils.populate(adminMap, map);
		
		// 注册日期类型转换器
		ConvertUtils.register(new DateLocaleConverter(), Date.class);
		// 把表单提交的数据，封装到对象中
        BeanUtils.copyProperty(admin, "birth", birth);
		// 测试
		System.out.println(admin);
	}

```
<br/>
**2.元数据**

（1）定义：可以用元数据获取数据库、表、列的定义信息。

（2）在jdbc中： 数据库元数据、参数元数据、结果集元数据

例1：获取数据库元数据

```
public void testDB() throws Exception {
		// 获取连接
		Connection conn = JdbcUtil.getConnection();
		// 获取数据库元数据
		DatabaseMetaData metaData = conn.getMetaData();
		
		System.out.println(metaData.getUserName());
		System.out.println(metaData.getURL());
		System.out.println(metaData.getDatabaseProductName());
	}

```

例2：获取参数元数据

```
public void testParams() throws Exception {
		// 获取连接
		Connection conn = JdbcUtil.getConnection();
		// SQL
		String sql = "select * from dept where deptid=? and deptName=?";
		// Object[] values = {"tom","888"};
		
		PreparedStatement pstmt = conn.prepareStatement(sql);
		// 参数元数据
		ParameterMetaData p_metaDate = pstmt.getParameterMetaData();
		// 获取参数的个数
		int count = p_metaDate.getParameterCount();
		
		
		// 测试
		System.out.println(count);
	}

```

例3：获取结果元数据

```
public void testRs() throws Exception {
		String sql = "select * from dept ";
		
		// 获取连接
		Connection conn = JdbcUtil.getConnection();
		PreparedStatement pstmt = conn.prepareStatement(sql);
		ResultSet rs = pstmt.executeQuery();
		// 得到结果集元数据(目标：通过结果集元数据，得到列的名称)
		ResultSetMetaData rs_metaData = rs.getMetaData();
		
		// 迭代每一行结果
		while (rs.next()) {
			// 1. 获取列的个数
			int count = rs_metaData.getColumnCount();
			// 2. 遍历，获取每一列的列的名称
			for (int i=0; i<count; i++) {
				// 得到列的名称
				String columnName = rs_metaData.getColumnName(i + 1);
				// 获取每一行的每一列的值
				Object columnValue = rs.getObject(columnName);
				// 测试
				System.out.print(columnName + "=" + columnValue + ",");
			}
			System.out.println();
		}
		
	}

```
<br/>

**3.DbUtils组件**

（1）定义：
commons-dbutils 是 Apache 组织提供的一个开源 JDBC工具类库，它是对JDBC的简单封装，学习成本极低，并且使用dbutils能极大简化jdbc编码的工作量，同时也不会影响程序的性能。

（2）作用：DbUtils组件能简化jdbc操作

（3）QueryRunner   组件的核心工具类：
定义了所有的与数据库操作的方法(查询、更新)

（4）常用方法：
	
* Int  update(Connection conn, String sql, Object param);   执行更新带一个占位符的sql
* Int  update(Connection conn, String sql, Object…  param); 执行更新带多个占位符的sql
* Int[]  batch(Connection conn, String sql, Object[][] params)        批处理
* T  query(Connection conn ,String sql, ResultSetHandler<T> rsh, Object... params)   查询方法
、
* Int  update( String sql, Object param);  
* Int  update( String sql, Object…  param); 
* Int[]  batch( String sql, Object[][] params)       

注意： 如果调用DbUtils组件的操作数据库方法，没有传入连接对象，那么在实例化QueryRunner对象的时候需要传入数据源对象
例： QueryRunner qr = new QueryRunner(ds);

（5）DbUtils提供的封装结果的对象：

* BeanHandler: 查询返回单个对象
* BeanListHandler: 查询返回list集合，集合元素是指定的对象
* ArrayHandler, 查询返回结果记录的第一行，封装对对象数组, 即返回：Object[]
*  ArrayListHandler, 把查询的每一行都封装为对象数组，再添加到list集合中
*  ScalarHandler 查询返回结果记录的第一行的第一列  (在聚合函数统计的时候用)
*  MapHandler  查询返回结果的第一条记录封装为map


<br/>
<br/>
本人才疏学浅，若有错误，请指出
谢谢！