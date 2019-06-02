###**引入**

* 在上一篇文章中，我介绍了Linux的基本知识，现在就带领你入门了，我认为Linux就是命令行的艺术，所以，我们不得不熟练掌握Linux的常用命令，下面我来详细介绍。

* 我在这声明一下，我使用的虚拟机是VMware WorkStation的12.1.0版本，Linux系统是Ubuntu 14.04

###**文件相关**

|序号|命令|功能|
|:------|:-----|:------|
|1|pwd|显示当前路径|
|2|cat|打开文件|
|3|tac|逆序打开文件|
|4|more|查看全部文件（只能向后翻页）|
|5|less|查看全部文件（可向前向后翻页）|
|6|nl|打开文件并显示行号|
|7|mkdir|创建文件夹|
|8|rm|删除文件或文件夹|
|9|cd|改变所在目录|
|10|cp|复制文件|
|11|mv|移动文件|
|12|touch|创建文件|
|13|gzip|压缩文件|
|14|gunzip|解压文件|
|15|which|查看可执行文件的位置|
|16|whereis|搜索程序名|
|17|date|显示日期|
|18|ls|查看目录的内容|
|19|wc|显示文件的行数、词数、字节数|
|20|head|默认显示文件的前10行|
|21|tail|显示文件末尾内容|


**详细介绍如下：**

**1.pwd**（print working directory）：显示当前路径

![这里写图片描述](http://img.blog.csdn.net/20170701163809344?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.cat**（Concatenate 串联）打开文件

![这里写图片描述](http://img.blog.csdn.net/20170701182227517?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.tac**：逆序打开文件

![这里写图片描述](http://img.blog.csdn.net/20170701182549429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**4.more**：查看全部文件（只能向后翻页）

|参数| 功能|
|:------|:------|
|+n      从笫n行开始显示|
|-n   |    定义屏幕大小为n行|
|+/pattern| 在每个档案显示前搜寻该字串（pattern），然后从该字串前两行之后开始显示 |
|-c      | 从顶部清屏，然后显示|
|-d       |提示“Press space to continue，’q’ to quit”，禁用响铃功能|
|-l      |  忽略Ctrl+l（换页）字符|
|-p     |  通过清除窗口而不是滚屏来对文件进行换页，与-c选项相似|
|-s     |  把连续的多个空行显示为一行|
|-u   |    把文件内容中的下画线去掉|

**常用操作**

|操作  |作用|
|:------|:------|
|Enter|    向下n行，需要定义。默认为1行|
|Ctrl+F|   向下滚动一屏|
|空格键 | 向下滚动一屏|
|Ctrl+B  |返回上一屏|
|= |      输出当前行的行号|
|：f |    输出文件名和当前行的行号|
|V|     调用vi编辑器|
|!|   调用Shell，并执行命令|
|q |      退出more|

**5.less**：查看全部文件（可向前向后翻页）

|参数|功能|
|:-----|:-----|
|-i | 忽略搜索时的大小写|
|-N|  显示每行的行号|
|-o|  <文件名> 将less 输出的内容在指定文件中保存起来|
|-s | 显示连续空行为一行|
|/字符串：|向下搜索“字符串”的功能|
|?字符串：|向上搜索“字符串”的功能|
|n：|重复前一个搜索（与 / 或 ? 有关）|
|N：|反向重复前一个搜索（与 / 或 ? 有关）|
|-x <数字>| 将“tab”键显示为规定的数字空格|
|b | 向后翻一页|
|d | 向后翻半页|
|h | 显示帮助界面|
|Q | 退出less 命令|
|u|  向前滚动半页|
|y | 向前滚动一行|
|空格键 |滚动一行|
|回车键 |滚动一页|
|[pagedown]| 向下翻动一页|
|[pageup]| 向上翻动一页|

**6.nl**：（Number of Lines）打开文件并显示行号

![这里写图片描述](http://img.blog.csdn.net/20170701191622253?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**7.mkdir**（make directories）创建文件夹

* –p 递归创建多个文件夹

![这里写图片描述](http://img.blog.csdn.net/20170701183331541?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**8.rm**（remove）：删除文件或文件夹

* -f   强制删除。忽略不存在的文件，不提示确认
*  -r   递归删除目录及其内容

![这里写图片描述](http://img.blog.csdn.net/20170701192013028?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**9.cd**（change directory）改变所在目录

* cd  ~     转到/home/用户  目录下
* cd  /      转到根目录中  
* cd  /home/qingaolei/Dream 转到Dream目录（绝对路径 ）
* cd  ../blog 转到上层目录下的blog子目录中（相对路径）
* cd ..返回上一级目录

实例：

![这里写图片描述](http://img.blog.csdn.net/20170701172925146?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**10.cp**（copy）：复制文件

![这里写图片描述](http://img.blog.csdn.net/20170701192601674?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**11.mv**（move）：移动文件

* 既可以移动文件也可以更改文件名称

![这里写图片描述](http://img.blog.csdn.net/20170701193050821?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**12.touch**：创建文件

![这里写图片描述](http://img.blog.csdn.net/20170701193334152?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**13.gzip**：压缩文件

**14.gunzip**：解压文件

![这里写图片描述](http://img.blog.csdn.net/20170701194459967?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**15.which**：查看可执行文件的位置

![这里写图片描述](http://img.blog.csdn.net/20170701190409793?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**16.whereis**：搜索程序名

* 只搜索二进制文件（-b）、man说明文件（-m）和源代码文件（-s）。默认返回所有信息

![这里写图片描述](http://img.blog.csdn.net/20170701190624954?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**17.date**：显示日期

|格式|	说明|
|:-----|:-----|
|%a|	星期几的简称，例如一、二、三|
|%A|	星期几的全名，例如星期一、星期二|
|%D	|日期(mm/dd/yy格式)|
|%F |  日期（yy-mm-dd格式）|
|%T	|显示时间格式，24小时制(hh:mm:ss)|
|%x|	显示日期的全称格式(mm/dd/yy)|
|%y|	年的最后两个数字|
|%Y	|年(如2007、2008)|
|%r	|时间(hh:mm:ss 上午或下午)|
|%p|	显示上午或下午|

实例

![这里写图片描述](http://img.blog.csdn.net/20170701164856609?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注： date命令由localtime（） 函数将时间数值转换为日期，但localtime函数不能同时使用，它是非线程安全（静态变量）。 localtime_r则是线程安全的，strftime也是线程安全的。


**18.ls**（list）：查看目录的内容

|参数|含义|
|:-----|:----|
|-a	|列举目录中的全部文件，包括隐藏文件|
|-l	|列举目录中的细节，依次为权限、链接数、所有者、组群、大小、创建日期、文件名|
|-i|列举文件的节点号|
|-f	|列举的文件显示文件类型|
|-r|	逆向，从后向前地列举目录中内容|
|-s|	大小，按文件大小排序|
|-h	|以人类可读的方式显示文件的大小，如用K、M、G作单位|

实例

![这里写图片描述](http://img.blog.csdn.net/20170701162620876?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**注：命令 ls -a -l 等价于 ll**


**19.wc**（Word Count ）：显示文件的行数、词数、字节数

* -l 统计行数。
* -w 统计字数
* -c 统计字节数

![这里写图片描述](http://img.blog.csdn.net/20170701195324420?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**20.head**：默认显示文件的前10行

* 参数 -n 表示显示文件的前n行

![这里写图片描述](http://img.blog.csdn.net/20170701185330495?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**21.tail**：显示文件末尾内容

* 参数 -n 表示显示最后n行（从后向前）

![这里写图片描述](http://img.blog.csdn.net/20170701185748519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



###**权限相关命令**

|序号|命令|功能|
|:------|:-----|:------|
|1|chmod|改变文件的权限|
|2|chgrp|改变文件所属组的权限|
|3|chown|改变文件所属者的权限|
|4|whoami|查看当前用户|
|5|who|显示当前登入系统的用户信息|
|6|su|切换用户|
|7|useradd|添加用户|


**详细介绍如下：**

**1.chmod**（change mode）：改变文件的权限

（1）权限

* r—文件可以被读取 
* w—文件可以被写入 
* x—文件可以被执行

也可以用数字表示权限：4——读取，2——写入，1——执行

（2）chmod 文件的使用者(u,g,o,a)增减(+,-,=)权限名称(r,w,x) 文件

（3）等价命令，但用数字表示更简单，所以我一般用数字表示。

chmod 751 re.txt 等价于 chmod u+rwx,g=rx,o=x re.txt


实例：

![这里写图片描述](http://img.blog.csdn.net/20170701170622562?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.chgrp**（Change group）：改变文件的所属群组

![这里写图片描述](http://img.blog.csdn.net/20170701200152103?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.chown**（Change owner）：改变文件拥有者

![这里写图片描述](http://img.blog.csdn.net/20170701200326095?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


**4.whoami**：查看当前用户

例：见 7.useradd：添加用户的图

**5.who**：显示当前登入系统的用户信息

例：见 7.useradd：添加用户的图

**6.su**:（Substitute User ）切换用户

![这里写图片描述](http://img.blog.csdn.net/20170701201226772?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**7.useradd**：添加用户

![这里写图片描述](http://img.blog.csdn.net/20170701201758737?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



<br>
由于篇幅有限，还有一部分基本命令，我就放到下一篇博文讲解！

<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文

