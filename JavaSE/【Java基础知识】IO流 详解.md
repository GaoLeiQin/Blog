**1.概念**

（1）io流用来处理设备之间的数据传输；
（2）Java对数据的操作的操作是通过流的方式；
（3）Java用于操作流的对象都在IO包；
（4）io流按操作数据分为两种：字节流和字符流；
（5）io流按流向分为：输入流、输出流

<font color="blue">Java流类图结构：</font>

![这里写图片描述](http://img.blog.csdn.net/20170413204035082?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
**2.字节流**

（1）定义：
用于读取二进制文件及任何类型文件

（2）InputStream类型

<font color="blue">InputStream流图结构：</font>
![这里写图片描述](http://img.blog.csdn.net/20170413204239336?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
*ByteArrayInputStream      允许将内存中的缓冲区当作InputStream使用
*StringBufferInputStream  将String转换成InputStream
*FileInputStream                用于从文件中读取信息
*PipedInputStream  产生用于写入相关PipedOutputStream的数据，实现管道化概念
*SequenceInputStream 将两个或多个InputStream对象转换成单一InputStream
*FilterInputStream  抽象类作为装饰器的接口
<br/>
（2）OutputStream类型

<font color="blue">OutputStream流图结构：</font>
![这里写图片描述](http://img.blog.csdn.net/20170413204401630?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br/>
*ByteArrayOutputStream  在内存中创建缓冲区，所有送往流的数据都要放置在缓冲区
*FileOutputStream  用于将信息写入文件
*PipedOutputStream 任何写入其中的信息都会自动作为相关PipedInputStream的输出
*FilterOutputStream 抽象类作为装饰器的接口
<br/>
**3.字符流**

（1）定义：
用于读写文件文本，不能操作二进制文件。

<font color="blue">字符流图结构：</font>
![这里写图片描述](http://img.blog.csdn.net/20170413204650826?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）Reader类型

（3）Writer类型

<br/>
**4.文件输入与输出**

（1）字节的输入与输出

*从键盘输入

例：

```java
InputStreamReader in = new InputStreamReader(System.in);
```
*从文件中读取

例：

```java
InputStreamReader in = new InputStreamReader(new FileInputStream("dream.txt"));
```

（2）字符的输入与输出

*创建一个文件对象

例：
```java
FileWriter f = new FileWriter("F:\\BAT\\writefile.txt");
```

*读取文件对象

例：

```java
FileReader fr = new FileReader("dream.txt");
```

*输出文本格式

例：
```
PrintWriter pw = new PrintWriter("possible.txt");
```
*在缓冲区输入文件

例：

```
BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
```

*格式化的内存输入

例：

```
DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream("red.txt")));
```
<br/>
**5.复制文件**

（1）字符流方式

例：

```
public class CopyFile {

    public static void main(String[] args) {
        //文件读(输入流)字符对象
        FileReader f = null;
        //文件写（输出流）字符对象
        FileWriter f1 = null;

        try {
            f = new FileReader("F:\\学习\\资料\\file1.txt");
            f1 = new FileWriter("F:\\学习\\资料\\writefile1.txt");
            //n记录实际读取到的字符数
            int n = 0;
            //读入到内存
            char[] c = new char[1024];
            try {
                while ((n = f.read(c)) != -1){
                    f1.write(c,0,n);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }


        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                f.close();
                f1.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

（2）字节流方式

例：

```
public class CopyPicture {

    public static void main(String[] args) {
        //先把图片读入内存，再写入到某个文件
        //因为是二进制文件，因此只能用字节流完成

        //输入流
        FileInputStream f = null;
        //输出流
        FileOutputStream f1 = null;
        try {
            f = new FileInputStream("F:\\学习\\资料\\image\\background_lawn.jpg");
            f1 = new FileOutputStream("F:\\学习\\资料\\copybackground.jpg");
            byte[] bytes = new byte[1024];
            //n记录实际读取到的字节数
            int len = 0;
            //循环读1
                while ((len=f.read(bytes)) != -1){
                    f1.write(bytes,0,len);
                }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {

            try {
                f.close();
                f1.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```
<br/>
**6.对象流与序列化**

（1）对象流
*操作对象：ObjectInputStream与ObjectOutputStream
被操作的对象需要实现serializable接口（没有方法的接口，即标记接口）

（2）序列化：
每个对象都用一个序列号保存

（3）意义： 
序列化用序列号代替了内存地址。

（4）文件格式：
对象序列化以特殊的文件格式存储对象数据

（5）修改默认的序列化机制
*某些数据域不可被序列化
*将它们标记成是transient（瞬时），因为瞬时的域可以防止序列化。

<br/>
<br/>
本人才疏学浅，如有错误，请指出~ 
谢谢！