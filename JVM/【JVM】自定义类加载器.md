**1.ClassLoader类介绍**

（1）	作用：根据一个指定类的名称，找到或者生成其对应的字节代码，从这些代码中定义出一个Java类。即Java.long.Class类的一个实例，还加载Java应用所需的资源，如图像文件、配置文件等。

（2）	常用方法：

* getParent()   返回该类加载器的父类加载器

* loadClass(String name) 负责加载指定的类，首先会从已加载的类中去寻找，若没有，从ClassLoader中加载，否则从Bootstrap中加载，否则抛出
ClassNotFoundException。

* defineClass(String name, byte[] b, int off, int len) 
          将一个 byte 数组转换为 Class 类的实例。

* findClass(String name) 
          使用指定的二进制名称查找类

* getPackages() 
          返回此类加载器及其祖先所定义的所有 Package

* getSystemClassLoader() 
          返回委托的系统类加载器

**2.自定义类加载器**

（1）流程：

* 继承Java.long.ClassLoader

* 检查请求的类型是否已经被这个类加载器加载到命名空间中了，若已装载，直接返回

* 委派类加载器请求给父类加载器，若父类加载器能够完成，则返回父类加载器加载的Class实例

* 调用本类加载器的findClass方法，试图获取对应的字节码，若获取到，则调用defineClass方法导入类型到方法区，否则抛出异常，终止类的加载过程

注意：被两个类加载器加载的同一个类，JVM认为是不同的类。

**流程图：**

![这里写图片描述](http://img.blog.csdn.net/20170705190456800?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


(2)	分类：

文件类加载器、网络类加载器、加密解密类加载器（取反操作，DES对称加密解密）

（3）实例

例1：文件类加载器

```
public class MyFileClassLoader extends ClassLoader{

    private String rootDir;

    public MyFileClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Class<?> c = findLoadedClass(name);

        //先查询是否已经加载了该类,若加载，直接返回，否则加载新的类
        if (c != null) {
            return c;
        }else {
            ClassLoader parent = this.getParent();
            try {
                c = parent.loadClass(name); //委派给父类加载
            }catch (Exception e){
//                e.printStackTrace();
            }

            if (c != null) {
                return c;
            }else {
                byte[] classData = getClassData(name);
                if (classData == null) {
                    throw new ClassNotFoundException();
                }else {
                    c = defineClass(name,classData,0,classData.length);
                }
            }
        }
        return c;
    }

    private byte[] getClassData(String classname) {
        String path = rootDir+"/"+classname.replace(".","/")+".class";

        InputStream is = null;
        ByteArrayOutputStream boas = new ByteArrayOutputStream();
        try {
            is = new FileInputStream(path);


            byte[] buffer = new byte[1024];
            int temp = 0;
            while ((temp = is.read(buffer)) != -1) {
                boas.write(buffer,0,temp);
            }

            return boas.toByteArray();

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (is != null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        //用工具类将流中的数据转换为字节数组
        try {
            IOUtils.toByteArray(path);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;

    }
}


class Load {
    public static void main(String[] args) throws ClassNotFoundException {
        MyFileClassLoader loader = new MyFileClassLoader("G:/MyData");
        MyFileClassLoader loader2 = new MyFileClassLoader("G:/MyData");

        Class<?> c = loader.loadClass("jvm.MyData");
        Class<?> c2 = loader.loadClass("jvm.MyData");
        Class<?> c3 = loader2.loadClass("jvm.MyData");

        Class<?> c4 = loader2.loadClass("java.lang.String");
        Class<?> c5 = loader2.loadClass("jvm.A");

        //查看是不是同一个类
        System.out.println(c.hashCode());
        System.out.println(c2.hashCode());
        System.out.println(c3.hashCode());
        //查看类加载器类型
        System.out.println(c3.getClassLoader());
        System.out.println(c4.getClassLoader());
        System.out.println(c5.getClassLoader());

    }
}
```

输出结果：

```
1523554304
1523554304
1175962212
jvm.MyFileClassLoader@2c7b84de
null
sun.misc.Launcher$AppClassLoader@18b4aac2
```

例2：网络类加载器

只需将例1 中的getClassData方法的代码改为

```
    private byte[] getClassData(String classname) {
        String path = rootDir+"/"+classname.replace(".","/")+".class";

        InputStream is = null;
        ByteArrayOutputStream boas = new ByteArrayOutputStream();
        try {
            is = new FileInputStream(path);

            //网络类加载器
            URL url =  new URL(path);
            is = url.openStream();

            byte[] buffer = new byte[1024];
            int temp = 0;
            while ((temp = is.read(buffer)) != -1) {
                boas.write(buffer,0,temp);
            }

            return boas.toByteArray();

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (is != null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        //用工具类将流中的数据转换为字节数组
        try {
            IOUtils.toByteArray(path);
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;

    }

```

例3：加密解密类加载器（取反操作）

加密方法：
```
    //加密方法（取反）
    public static void encrpt(String src, String dest) {
        FileInputStream fis = null;
        FileOutputStream fs = null;

        try {
            fis = new FileInputStream(src);
            fs = new FileOutputStream(dest);

            int temp = 0;

            while ((temp = fis.read()) != -1) {
                fs.write(temp^0xff); //取反操作
            }

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fs != null) {
                try {
                    fs.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

```

解密：

```
public class MySecureLoader extends ClassLoader{

    public static void main(String[] args) throws ClassNotFoundException {
        MySecureLoader de = new MySecureLoader("G:\\mydata");

        Class<?> c = de.findClass("jvm.MyData");
        System.out.println(c);
    }


    String rootDir ;
    public MySecureLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Class<?> c = findLoadedClass(name);

        //先查询是否已经加载了该类,若加载，直接返回，否则加载新的类
        if (c != null) {
            return c;
        }else {
            ClassLoader parent = this.getParent();
            try {
                c = parent.loadClass(name); //委派给父类加载
            }catch (Exception e){
//                e.printStackTrace();
            }

            if (c != null) {
                return c;
            }else {
                byte[] classData = getClassData(name);
                if (classData == null) {
                    throw new ClassNotFoundException();
                }else {
                    c = defineClass(name,classData,0,classData.length);
                }
            }
        }
        return c;
    }

    private byte[] getClassData(String classname) {
        String path = rootDir+"/"+classname.replace(".","/")+".class";

        InputStream is = null;
        ByteArrayOutputStream boas = new ByteArrayOutputStream();
        try {
            is = new FileInputStream(path);

            int temp = 0;

            while ((temp = is.read()) != -1) {
                boas.write(temp^0xff); //取反操作
            }

            return boas.toByteArray();

        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (is != null){
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
}


```

输出结果：class jvm.MyData

分析：从结果可以看出，解密成功

<br>
<br>

本人才疏学浅，若有错，请指出
谢谢！