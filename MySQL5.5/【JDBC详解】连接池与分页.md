##一、连接池

**1.由来：**

一个用户至少要用到一个连接。当用户过多时，需要创建巨大数量的连接对象，这会使数据库承受极大的压力，为了解决这种现象，出现了数据库连接池。

**2.定义：**

在用户和数据库之间创建一个”池”，这个池中有若干个连接对象，当用户想要连接数据库，就要先从连接池中获取连接对象，然后操作数据库。即数据库连接池就是提供连接的

![这里写图片描述](http://img.blog.csdn.net/20170429214703269?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
**3.自定义连接池**

例：

```
public class MyPool {
    private int init_count = 3;		// 初始化连接数目
    private int max_count = 6;		// 最大连接数
    private int current_count = 0;  // 记录当前使用连接数
    // 连接池 （存放所有的初始化连接）
    private LinkedList<Connection> pool = new LinkedList<Connection>();


    //1.  构造函数中，初始化连接放入连接池
    public MyPool() {
        // 初始化连接
        for (int i=0; i<init_count; i++){
            // 记录当前连接数目
            current_count++;
            // 创建原始的连接对象
            Connection con = createConnection();
            // 把连接加入连接池
            pool.addLast(con);
        }
    }

    //2. 创建一个新的连接的方法
    private Connection createConnection(){
        try {
            Class.forName("com.mysql.jdbc.Driver");
            // 原始的目标对象
            final Connection con = DriverManager.getConnection("jdbc:mysql:///test", "root", "965847");

            // 对con创建其代理对象
            Connection proxy = (Connection) Proxy.newProxyInstance(

                    con.getClass().getClassLoader(),    // 类加载器
                    //con.getClass().getInterfaces(),   // 当目标对象是一个具体的类的时候
                    new Class[]{Connection.class},      // 目标对象实现的接口

                    new InvocationHandler() {
                        @Override
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                            // 方法返回值
                            Object result = null;
                            // 当前执行的方法的方法名
                            String methodName = method.getName();

                            // 判断当执行了close方法的时候，把连接放入连接池
                            if ("close".equals(methodName)) {
                                System.out.println("begin:当前执行close方法开始！");
                                // 连接放入连接池
                                pool.addLast(con);
                                System.out.println("end: 当前连接已经放入连接池了！");
                            } else {
                                // 调用目标对象方法
                                result = method.invoke(con, args);
                            }
                            return result;
                        }
                    }
            );
            return proxy;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    //3. 获取连接
    public Connection getConnection(){

        // 3.1 判断连接池中是否有连接, 如果有连接，就直接从连接池取出
        if (pool.size() > 0){
            return pool.removeFirst();
        }

        // 3.2 连接池中没有连接： 判断，如果没有达到最大连接数，创建；
        if (current_count < max_count) {
            // 记录当前使用的连接数
            current_count++;
            // 创建连接
            return createConnection();
        }

        // 3.3 如果当前已经达到最大连接数，抛出异常
        throw new RuntimeException("当前连接已经达到最大连接数目 ！");
    }


    //4. 释放连接
    public void realeaseConnection(Connection con) {
        // 4.1 判断： 池的数目如果小于初始化连接，就放入池中
        if (pool.size() < init_count){
            pool.addLast(con);
        } else {
            try {
                // 4.2 关闭
                current_count--;
                con.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
    }

    public static void main(String[] args) throws SQLException {
        MyPool pool = new MyPool();
        System.out.println("当前连接: " + pool.current_count); 

        // 使用连接
        pool.getConnection();
        pool.getConnection();
        Connection con3 = pool.getConnection();
        Connection con2 = pool.getConnection();
        Connection con1 = pool.getConnection();

        // 释放连接, 连接放回连接池
        con1.close();

        // 再获取
        pool.getConnection();

        System.out.println("连接池：" + pool.pool.size());  
        System.out.println("当前连接: " + pool.current_count); 
    }

}
```
<br/>
**4.DBCP连接池**

（1）引入jar包：

•	Commons-dbcp.jar：连接池的实现
•	Commons-pool.jar：连接池实现的依赖库

（2）核心类：BasicDataSource

例：

```
public class Dbcp {
    // 1. 硬编码方式实现连接池
    @Test
    public void testDbcp() throws Exception {
        // DBCP连接池核心类
        BasicDataSource dataSouce = new BasicDataSource();
        // 连接池参数配置：初始化连接数、最大连接数 / 连接字符串、驱动、用户、密码
        dataSouce.setUrl("jdbc:mysql:///test");//数据库连接字符串
        dataSouce.setDriverClassName("com.mysql.jdbc.Driver");//数据库驱动
        dataSouce.setUsername("root");	   //数据库连接用户
        dataSouce.setPassword("965847");   //数据库连接密码
        dataSouce.setInitialSize(3);       // 初始化连接
        dataSouce.setMaxActive(6);	       // 最大连接
        dataSouce.setMaxIdle(3000);        // 最大空闲时间
  
        // 获取连接
        Connection con = dataSouce.getConnection();
        con.prepareStatement("delete from admin where id=5").executeUpdate();
        // 关闭
        con.close();
    }

    @Test
    // 2. 【推荐】配置方式实现连接池
    public void testProp() throws Exception {
        // 加载prop配置文件
        Properties prop = new Properties();
        // 获取文件流
        InputStream inStream = Dbcp.class.getResourceAsStream("db.properties");
        // 加载属性配置文件
        prop.load(inStream);
        // 根据prop配置，直接创建数据源对象
        DataSource dataSouce = BasicDataSourceFactory.createDataSource(prop);

        // 获取连接
        Connection con = dataSouce.getConnection();
        con.prepareStatement("delete from admin where id=4").executeUpdate();
        // 关闭
        con.close();
    }

}

```

<br/>
**5.C3P0连接池**

（1）优点：最常用的连接池技术！Spring框架，默认支持C3P0连接池技术！

（2）核心类：CombopooledDataSource 

例：

```
public class C3p0 {
    @Test
    //1. 硬编码方式，使用C3P0连接池管理连接
    public void testCode() throws Exception {
        // 创建连接池核心工具类
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        // 设置连接参数：url、驱动、用户密码、初始连接数、最大连接数
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/jdbc_demo");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setUser("root");
        dataSource.setPassword("root");
        dataSource.setInitialPoolSize(3);
        dataSource.setMaxPoolSize(6);
        dataSource.setMaxIdleTime(1000);

        // ---> 从连接池对象中，获取连接对象
        Connection con = dataSource.getConnection();
        // 执行更新
        con.prepareStatement("delete from admin where id=7").executeUpdate();
        // 关闭
        con.close();
    }

    @Test
    //2. XML配置方式，使用C3P0连接池管理连接
    public void testXML() throws Exception {
        // 创建c3p0连接池核心工具类
        // 自动加载src下c3p0的配置文件【c3p0-config.xml】
        ComboPooledDataSource dataSource = new ComboPooledDataSource();// 使用默认的配置

        // 获取连接
        Connection con = dataSource.getConnection();
        // 执行更新
        con.prepareStatement("delete from admin where id=4").executeUpdate();
        // 关闭
        con.close();

    }

}
```

<br/>
##二、分页

**1.优点：**

利于页面布局，且显示的效率高！

**2.要点：**

关键点：
（1）	分页SQL语句；
（2）	后台处理： dao/service/servlet/JSP

3.一个初步的例子

例：

（1）创建分页的属性及方法
```
public class PageBean<T> {
	private int currentPage = 1; // 当前页, 默认显示第一页
	private int pageCount = 4;   // 每页显示的行数(查询返回的行数), 默认每页显示4行
	private int totalCount;      // 总记录数
	private int totalPage;       // 总页数 = 总记录数 / 每页显示的行数  (+ 1)
	private List<T> pageData;       // 分页查询到的数据

	// 返回总页数
	public int getTotalPage() {
		if (totalCount % pageCount == 0) {
			totalPage = totalCount / pageCount;
		} else {
			totalPage = totalCount / pageCount + 1;
		}
		return totalPage;
	}

	public void setTotalPage(int totalPage) {
		this.totalPage = totalPage;
	}

	public int getCurrentPage() {
		return currentPage;
	}
	public void setCurrentPage(int currentPage) {
		this.currentPage = currentPage;
	}
	public int getPageCount() {
		return pageCount;
	}
	public void setPageCount(int pageCount) {
		this.pageCount = pageCount;
	}
	public int getTotalCount() {
		return totalCount;
	}
	public void setTotalCount(int totalCount) {
		this.totalCount = totalCount;
	}

	public List<T> getPageData() {
		return pageData;
	}
	public void setPageData(List<T> pageData) {
		this.pageData = pageData;
	}
}

```

（2）具体的分页操作

```
public class EmployeeDao {

    public void getAll(PageBean<Employee> pb) {

        //2. 查询总记录数;  设置到pb对象中
        int totalCount = this.getTotalCount();
        pb.setTotalCount(totalCount);

		//若当前页 <= 0;         当前页设置当前页为1;
		//若当前页 > 最大页数；  当前页设置为最大页数;
        if (pb.getCurrentPage() <=0) {
            pb.setCurrentPage(1);					    // 把当前页设置为1
        } else if (pb.getCurrentPage() > pb.getTotalPage()){
            pb.setCurrentPage(pb.getTotalPage());	// 把当前页设置为最大页数
        }

        //1. 获取当前页： 计算查询的起始行、返回的行数
        int currentPage = pb.getCurrentPage();
        int index = (currentPage -1 ) * pb.getPageCount();	// 查询的起始行
        int count = pb.getPageCount();						// 查询返回的行数


        //3. 分页查询数据;  把查询到的数据设置到pb对象中
        String sql = "select * from employee limit ?,?";

        try {
            // 得到Queryrunner对象
            QueryRunner qr = JdbcUtils.getQueryRuner();
            // 根据当前页，查询当前页数据(一页数据)
            List<Employee> pageData = qr.query(sql, new BeanListHandler<Employee>(Employee.class), index, count);
            // 设置到pb对象中
            pb.setPageData(pageData);

        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }

    public int getTotalCount() {
        String sql = "select count(*) from employee";
        try {
            // 创建QueryRunner对象
            QueryRunner qr = JdbcUtils.getQueryRuner();
            // 执行查询， 返回结果的第一行的第一列
            Long count = qr.query(sql, new ScalarHandler<Long>());
            return count.intValue();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

```

<br/>
<br/>
本人才疏学浅，若有错误，请指出
谢谢！