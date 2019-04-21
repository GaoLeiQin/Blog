####**1.java中创建对象的五种方式**

* 用new语句创建对象，这是最常见的创建对象的方法。
* 运用反射手段,调用java.lang.Class或者java.lang.reflect.Constructor类的newInstance()实例方法。
* 调用对象的clone()方法。
* 通过I/O流（包括反序列化），

注：前两种会调用构造函数，而后两种不会调用构造函数。

####**2.使用new关键字**

最常见也是最简单的创建对象的方式。可以调用任意的构造函数(无参的和带参数的)。

例：

```
Book book = new Book();
```

####**3.使用Class类的newInstance方法**

```
class Book {

    private String name;

    public Book() {
        
    }
    
    
    public static void main(String[] args) throws Exception {
        //第一种方法
        Book book3 = (Book) Class.forName("other.Book").newInstance();
        System.out.println(book3);
        //第二种方法
        book3 = Book.class.newInstance();
    }
}
```

####**4.使用Constructor类的newInstance方法**

```
class Book {

    private String name;

    public Book() {

    }


    public static void main(String[] args) throws Exception {
        Constructor<Book> constructor = Book.class.getConstructor();
        Book book3 = constructor.newInstance();
    }
}
```

####**5.使用object.clone()**

如果要调用clone方法，那么该object需要实现Cloneable接口，并重写clone()方法。

```
class Book1 implements Cloneable {

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return (Book) super.clone();
    }

    public static void main(String[] args) {
        Book1 book = new Book1();
        Book1 bookclone = book.clone();
    }

}
```

####**6.使用反序列化**

```
class Book implements Serializable {

    private String name;

    public Book() {

    }

    public static void main(String[] args) throws Exception {
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("Book.java"));
        Book emp5 = (Book) in.readObject();
    }
}
```


<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！
