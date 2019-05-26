###**引入**

class文件就是字节码文件，是由虚拟机执行的文件。也就是java语言和C & C++语言的区别就是，整个编译执行过程多了一个虚拟机这一步。这个在  [类文件结构](http://blog.csdn.net/baiye_xing/article/details/73734384)  中已经解释。上一节讲了虚拟机是如何加载一个class的，这一节就讲解虚拟机是如何执行class文件的。


###**运行时栈帧结构**

**1.定义**

* 栈是每个线程独有的内存。

* 栈帧存储了局部变量表，操作数栈，动态连接，和返回地址等。

* 每一个方法的执行 对应的一个栈帧在虚拟机里面从如栈到出栈的过程。

* 只有位于栈顶的栈帧才有有效的，对应的方法称为当前方法。

* 执行引擎运行的所有指令只针对当前栈帧和当前方法。

**2.栈帧结构**

* 每一个栈帧都包含了局部变量表、操作数栈、动态连接、方法返回地址和一些额外的附加信息。

* 在编译 程序代码的时候，栈帧中需要多大的局部变量表、多深的操作数栈都已经完全确定了，并且写入到方法表的Code属性之中，因此一个栈帧需要分配多少内存，不会受到程序运行期变量数据的影响，而仅仅取决于具体的虚拟机实现。

**栈帧图解：**

![这里写图片描述](http://img.blog.csdn.net/20170626160451840?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.局部变量表**

（1）定义：

* 局部变量表存放的一组变量的存储空间。存放方法参数和方法内部定义的局部变量表。

* 在java编译成class的时候，已经确定了局部变量表所需分配的最大容量。

* 局部变量表的最小单位是一个Slot。

* 虚拟机规范没有明确规定一个Slot占多少大小。只是规定，它可以放下boolean,byte,...reference &return address.

* reference 是指一个对象实例的引用。关于reference的大小，目前没有明确的指定大小。但是我们可以理解为它就是类似C++中的指针。

* 局部变量表的读取方式是索引，从0开始。所以局部变量表可以简单理解为就是一个表.

（2）局部变量表的分配顺序如下：

* this 引用（隐式参数）

* 方法的参数表。

* 根据局部变量顺序，分配Solt。

* 一个变量一个solt，64为的占2个solt。java中明确64位的是long & double

注意：局部变量只给予分配的内存，没有class对象的准备阶段，所以局部变量在使用前，必须先赋值（无初始值）。为了尽可能的节约局部变量表，Solt可以重用。


**4.操作数栈**

用来存放操作数的栈结构，当一个方法刚开始执行的时候，这个方法的操作数栈是空的，在方法的执行过程中，会有各种字节码指令向操作数栈中写入和提取内容，也就是入栈和出栈的操作。

注：java虚拟机的解释执行引擎称为基于栈的执行引擎，其中所指的栈就是操作数栈

**5.动态连接**

* 每个栈帧都包含一个指向运行时常量池的引用，持有这个引用是为了支持动态连接。

* 符号池的引用，有一部分是在第一次使用或者初始化的时候就确定下来，这个称为静态引用。

* 还有一部分是在每一次的运行期间转化为直接引用，这个就是动态连接


**6.方法返回地址**

（1）方法被执行后，有两种方式退出这个方法

* 执行引擎遇到任意一个方法的返回的字节码指令（如return）
* 在方法执行过程中遇到了异常，并且这个异常并没有在方法体中得到处理。

（2）方法退出之后，需要返回到方法被调用的位置，程序才能继续执行，方法返回时需要在栈帧中保存一些信息，用以帮助它恢复它上层方法的执行状态。一般情况下，调用者的pc计数器的值可以作为返回地址，栈帧中很可能会保存这个计数器值，方法异常退出时，返回地址是要通过异常处理器表来确定，栈帧中一般不会保存这部分信息。

（3）方法退出的过程实际上等同于把当前栈帧出栈，所以可能需要执行这些操作：恢复上层方法的局部变量表和操作数栈，把返回值压入调用者栈的操作数栈中，调整pc计数器的值。

注：附加信息：虚拟机规范允许具体的虚拟机实现增加一些规范里没有描述的信息到栈帧中，这部分信息取决于具体的虚拟机实现。

**7.方法调用**

（1）方法调用阶段的唯一任务就是确定被调用方法的版本（即调用哪一个方法），暂时不涉及方法内部的具体运行过程。

（2）class文件的编译过程中不包含传统编译中的连接步骤，一切方法调用在class文件中存储的都是符号引用，而不是方法在实际运行时内存布局中的入口地址，这使得java有着更强大的动态扩展能力，但也使得java方法的调用过程变得相对复杂起来，需要在类的加载甚至运行期间才能确定目标方法的直接引用。

（3）两种方法调用

* 解析调用：方法在程序真正运行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期是不可改变的

 * 符合这个条件的有静态方法，私有方法，实例构造器和父类方法四类，它们在类加载的时候会把符号引用解析为该方法的直接引用。

 * 解析调用一定是一个静态的过程，编译期间就完全确定，在类装载的解析阶段就会把涉及到的符号引用全部转化为可确定的直接引用，不会延迟到运行期间再去完成。

* 分派调用：可能是静态的也可能是动态的，根据分派依据的宗量数又可分为单分派和多分派。分派机制与java的多态机制关系密切。

 * 静态分派：依赖静态类型来定位方法执行版本的分派动作。静态分派的最典型的应用就是方法重载。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行的

 * 动态分派：在运行期间根据实际类型来确定方法执行版本的分派调用过程称为动态分派。这跟多态性的另一个体现——重写有着很密切的关联。

 * 单分派：根据一个宗量对目标方法进行选择

 * 多分派：根据多于一个的总量对目标方法进行选择。

###**常用字节码**

字节码指令是一个byte整数

**（1）	局部变量压栈**

* Xload（x为i l f d a）  a表示object ref

* Xload_n （n为 0 1 2 3）将第几个局部变量的值压栈

* Xaload（x为i f l d a b c s）从数组中取得给定索引的值，将该值压栈

**（2）	出栈装载入局部变量**

* Xstore （x为i l f d a）出栈，存入局部变量

* Xstore_n（n为0 1 2 3）出栈，将值存入第n个局部变量

* Xastore（x为i f l d a b c s）将值存入数组

**（3）	通用栈操作（无类型）**

* Nop

* Pop 弹出栈顶1个字长

* Dup 复制栈顶一个字长，复制内容压栈

**（4）	类型转化**

* I2l i2f l2i l2f l2d f2i f2d d2i d2l d2f i2b i2c i2s

* i2l表示将int转为long

**（5）	整数运算**

* Iadd lsub idiv imul linc

**（6）	浮点运算**

* Fadd dadd fsub fdiv ddiv fmul dmul

**（7）	对象操作指令**

* New getfield putfield getstatic putstatic 

**（8）	条件控制**

* Ifeq 若为0，则跳转，

* ifne 若不为0，则跳转

* iflt 若小于0，则跳转，

* ifge若大于0，则跳转

* if_icmpeq 如果两个int相同，则跳转

**（9）	方法调用**

* Invokevirtual

* Invokespecial

* Invokestatic

* Invokeinterface

* xreturn

**（10）	常量入栈**

* Aconst_null   null对象入栈

* Iconst_m1     int常量-1入栈

* Iconst_0       int常量0入栈

* Lconst_1       long常量1入栈

* Fconst_1       float常量1.0入栈

* Dconst_1       double常量1.0入栈

* Bipush         8位带符号整数入栈

* Sipush         16位带符号整数入栈

* Idc             常量池中的项入栈

说了这么多，下面来个例子实战一下

###**简单的字节码执行实例**

例：Java程序

```
public class TestFile {
	public int calc() {
		int a = 500;
		int b = 200;
		int c = 50;
		return (a + b) / c;
	}
}

```

执行 javap –verbose TestFile 命令生成反汇编代码如下：

```
public int testfile();
  Code:
   Stack=2, Locals=4, Args_size=1
   0:   sipush  500
   3:   istore_1
   4:   sipush  200
   7:   istore_2
   8:   bipush  50
   10:  istore_3
   11:  iload_1
   12:  iload_2
   13:  iadd
   14:  iload_3
   15:  idiv
   16:  ireturn
}

```
**执行过程**

1.将500入栈，然后出栈将500存入第一个局部变量

![这里写图片描述](http://img.blog.csdn.net/20170626163815399?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2.将200入栈，然后出栈将200存入第二个局部变量

![这里写图片描述](http://img.blog.csdn.net/20170626163824207?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3.将50入栈，然后将50存入第三个局部变量

![这里写图片描述](http://img.blog.csdn.net/20170626163836370?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4.将第一个局部变量的值（500）压栈，然后将第二个局部变量的值（200）也压栈

![这里写图片描述](http://img.blog.csdn.net/20170626163844004?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5.执行整型加法运算，然后将第三个局部变量的值（50）压栈

![这里写图片描述](http://img.blog.csdn.net/20170626163851519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

6.执行整型除法运算，最后执行整型的返回操作将值返回。

![这里写图片描述](http://img.blog.csdn.net/20170626163857197?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
<br>

本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文。