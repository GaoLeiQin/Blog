
###一个简单的java应用程序###

```java
public class FirstSample {
	public static void main(String[] args) {
		System.out.println("Hello,World");
	}
}
```

####一、命名规约：####
1.代码中的命名均不能以下划线或美元符号开始，也不能以下划线或美元符号结束。  

 反例：  _name    __name     $Object    name_     name$    Object$ 
 
2.代码中的命名严禁使用拼音与英文混合的方式，不允许直接使用中文。 

注意，即使纯拼音命名方式也要避免采用。  

反例： DaZhePromotion [打折] / getPingfenByName() [评分] / int 某变量 = 3 

正例： alibaba / taobao / youku / hangzhou 等国际通用的名称，可视同英文。

3.类名使用UpperCamelCase风格，必须遵从驼峰形式，但以下情形例外：

（领域模型的相关命名）DO / BO / DTO / VO等。 

正例：MarcoPolo / UserDO / XmlService / TcpUdpDeal / TaPromotion 

反例：macroPolo / UserDo / XMLService / TCPUDPDeal / TAPromotion

4.方法名、参数名、成员变量、局部变量都统一使用lowerCamelCase风格，必须
遵从驼峰形式。 

正例： localValue / getHttpMessage() / inputUserId

5. 常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字
长。 

正例： MAX_STOCK_COUNT 

反例： MAX_COUNT

6.抽象类命名使用Abstract或Base开头；异常类命名使用Exception结尾；测试类
命名以它要测试的类的名称开始，以Test结尾。

7.中括号是数组类型的一部分，数组定义如下：String[] args; 

反例：请勿使用String args[]的方式来定义。

8.包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一
使用单数形式，但是类名如果有复数含义，类名可以使用复数形式。 

正例： 应用工具类包名为com.alibaba.open.util、类名为MessageUtils

9.接口类中的方法和属性不要加任何修饰符号（public 也不要加），保持代码的简
洁性，并加上有效的Javadoc注释。尽量不要在接口里定义变量，如果一定要定义变量，肯定是与接口方法相关，并且是整个应用的基础常量。 

正例：接口方法签名：void f(); 接口基础常量表示：String COMPANY = "alibaba"; 

反例：接口方法定义：public abstract void f(); 

说明：JDK8中接口允许有默认实现，那么这个default方法，是对所有实现类都有价
值的默认实现。

10.枚举类名建议带上Enum后缀，枚举成员名称需要全大写，单词间用下划线隔开。 
说明：枚举其实就是特殊的常量类，且构造方法被默认强制是私有。 

正例：枚举名字：DealStatusEnum，成员名称：SUCCESS / UNKOWN_REASON。

####二、常量定义####
1.不允许出现任何魔法值（即未经定义的常量）直接出现在代码中。 

反例： String key="Id#taobao_"+tradeId； 

cache.put(key, value);

2.long或者Long初始赋值时，必须使用大写的L，不能是小写的l，小写容易跟数字1混淆，造成误解。 

说明：Long a = 2l; 写的是数字的21，还是Long型的2?

3.如果变量值仅在一个范围内变化用Enum类。如果还带有名称之外的延伸属性，必须使用Enum类，下面正例中的数字就是延伸信息，表示星期几。 

正例：public Enum{ MONDAY(1), TUESDAY(2), WEDNESDAY(3), THURSDAY(4), FRIDAY(5), SATURDAY(6), SUNDAY(7);}

####三、格式规约####
1.大括号的使用约定。如果是大括号内为空，则简洁地写成{}即可，不需要换行；如果是非空代码块则： 

1） 左大括号前不换行。 
2） 左大括号后换行。
3） 右大括号前换行。 
4） 右大括号后还有else等代码则不换行；表示终止右大括号后必须换行。

2.左括号和后一个字符之间不出现空格；同样，右括号和前一个字符之间也不出现空格。

3.if/for/while/switch/do等保留字与左右括号之间都必须加空格。

4.任何运算符左右必须加一个空格。 

说明：运算符包括赋值运算符=、逻辑运算符&&、加减乘除符号、三目运行符等。

5.缩进采用4个空格，不要使用tab字符。

```java
public static void main(String args[]) {
	// 缩进4个空格
	String say = "hello";
	// 运算符的左右必须有一个空格
	int flag = 0;
	// 关键词if与括号之间必须有一个空格，括号内的f与左括号，0与右括号不需要空格
	if (flag == 0) {
		System.out.println(say);
	}
	// 左大括号前加空格且不换行；左大括号后换行
	if (flag == 1) {
		System.out.println("world");
	// 右大括号前换行，右大括号后有else，不用换行
	} else {
		System.out.println("ok");
	// 在右大括号后直接结束，则必须换行
	}
}
```
6.单行字符数限不超过 120 个，超出需要换行时 个，超出需要换行时 遵循如下原则： 

1） 第二行相对一缩进 4个空格，从第三行开始不再继续缩进参考示例。 
2） 运算符与下文一起换行。 
3） 方法调用的点符号与下文一起换行。 
4） 在多个参数超长，逗号后进行换行。 
5） 在括号前不要换行，见反例。 

正例：
StringBuffer sb = new StringBuffer();

//超过120个字符的情况下，换行缩进4个空格，并且方法前的点符号一起换行
sb.append("zi").append("xin")...
.append("huang")...
.append("huang")...
.append("huang");

反例：
StringBuffer sb = new StringBuffer();
//超过120个字符的情况下，不要在括号前换行

sb.append("zi").append("xin")...append
("huang");

//参数很多的方法调用可能超过120个字符，不要在逗号前换行
method(args1, args2, args3, ...
, argsX);

7.方法参数在定义和传入时，多个参数逗号后边必须加空格。 
正例：下例中实参的"a",后边必须要有一个空格。
method("a", "b", "c")
</br>
</br>
本人才疏学浅，如有错误，请指出~ 
谢谢！