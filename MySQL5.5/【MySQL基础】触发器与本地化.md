## 触发器

1.定义：

为响应任意语句而自动执行的一条MySQL语句

2.创建触发器

```MySQl
create trigger newproduct after insert on products for each now select "product added";
```

3.删除触发器

```
drop trigger newproduct;
```

4.使用触发器

（1）insert触发器

```
create trigger neworder after insert on orders for each row select new.order_num;
```

（2）delete触发器

```
create trigger deleteorder before delete on orders for each row select new.order_num;
```

（3）update触发器

```
create trigger updateorder before update on vendors for each row set new.vend_state = upper(new.vend_state);
```

## 本地化

1.字符集

（1）查看字符集


![这里写图片描述](http://img.blog.csdn.net/20170420202635076?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）设置（控制台和输出结果）为中文

![这里写图片描述](http://img.blog.csdn.net/20170420203046801?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：设置后就解决了中文乱码问题，可以插入中文，结果显示也是中文。

2.更换控制台代码页命令（在）

（1）GBK  

```
chcp   936
```

如图所示：
![这里写图片描述](http://img.blog.csdn.net/20170420203825702?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）utf8 

```
 chcp  65001
```

（3）美国英语 

```
 chcp  437
```

<br/>
<br/>
本人水平有限，若有错，请指出~
谢谢！