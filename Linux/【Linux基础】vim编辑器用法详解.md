###**引入**

**1.vim概述**

* Linux下的编辑器最常用的就是vim或者vi文本编辑。vi 和vim 编辑器的区别是vim是vi的改进版本，在vi 编辑器的基础上上扩展了很多实用的功能。 大多数的linux/unix 发行版本中都使用 vim 代替了原来的 vi 文本编辑器

* vi/vim文本编辑器是我们在linux系统下工作可以说是必须会使用的一个工具，因为linux系统的设计理念是一切皆文件，也就是说，你在linux里的任何操作都是对文件的操作，所以会经常去操作文件，更改文件，保存文件，退出并保存文件。

**2.选择vim的原因**

Linux的命令行界面下面有非常多的文本编辑器。比如Emacs、pico、nano、joe与vim等，但是我为什么选择vim呢，听我给你娓娓道来：

* 所有的Unix like系统都会内置vi文本编辑器，其他的文本编辑器则不一定会存在。

* 很多软件的编辑接口都会主动调用vi。

* vim具有程序编辑的能力，可以主动以字体颜色辨别语法的正确性，方便程序设计。

* 程序简单，编辑速度快。

**3.vim界面**

![这里写图片描述](http://img.blog.csdn.net/20170703192710804?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**vim的三种模式概述**


在linux vi/vim 文本编辑器里有三种模式：命令模式，输入模式和末行模式。这三种模式分别代表什么含义呢？

**1.命令模式**

命令模式是vi/vim 编辑器进入后的默认模式，从命令模式可以切换到输入模式和编辑模式，

**2.输入模式**

输入模式（插入模式）是对文件做出更改操作。否则，你在命令模式下，vi/vim 文本编辑器是只读模式，你无法对文本做出更改。


**3.编辑模式**

编辑模式（末行模式）是是命令模式下输入”:”，当你在输入模式下，对文件做了更改，那么需要先退回到命令命令，再进入编辑模式，之后退出。

**4.图解三种模式**

![这里写图片描述](http://img.blog.csdn.net/20170703192755815?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


###**命令模式**

|快捷键|描述|
|:-----|:----|
|ZZ  |保存并退出|
|x| 删除光标所在处的字符|
|nx| 删除光标后的n个字符|
|dd| 删除光标所在的整行（剪切）|
|D |删除从光标所在处到行尾|
|n1,n2d|删除指定范围的行|
|yy / Y|复制当前行|
|nyy / nY| 复制当前行以下n行|
|p |粘贴在当前光标所在行下 |
|P(大写)|粘贴在当前光标所在行上|
|h/←|光标向左移动一个字符|
|j/↓|光标向下移动一个字符|
|k/↑|光标向上移动一个字符|
|l/→|光标向右移动一个字符|
|u|复原前一个操作|
|[Ctrl]+r|重做上一个操作|
|.小数点|重复前一个操作|
|Ctrl+F|向前翻整页|
|Ctrl+U|向前翻半页|
|Ctrl+B|向后翻整页|
|Ctrl+D|向后翻半页|
|gg|光标移到文件的第一行|
|G|光标移到文件的最后一行|
###**输入模式**

**进入输入模式的快捷键**

|快捷键|描述|
|:-----|:----|
|a |在光标后插入|
|A|在本行末尾插入|
|i|在光标前插入| 
|I|在本行开始插入|
|o|在光标下插入新行|
|O|在光标上插入新行|
|r|只替换光标所在那个字符一次|
|R|一直替换光标所在字符，直到按下Esc键|

###**编辑模式**

|快捷键|描述|
|:-----|:----|
|:set nu | 设置行号|
|:set nonu|  取消行号|
|:set hlsearch |设置高亮度查找|
|:set nohlsearch |取消高亮度查找|
|:set backup| 自动备份文件|
|:set ruler |开启右下角状态栏说明|
|:set showmode| 显示左下角的INSERT之类的状态栏|
|:set backspace={0,1,2}| 设置退格键功能。为2时可以删任意字符。为0或1时仅可以删除刚才输入的字符。|
|:set all |显示目前所有的环境参数值|
|:set |显示与系统默认值不同的参数值|
|:syntax on/off |是否开启依据相关程序语法显示不同的颜色|
|:set bg=dark/light| 是否显示不同的颜色色调|
|:/word|向下寻找一个名称为word的字符串。|
|:?word|向上寻找一个名称为word的字符串。|
|:n1,n2s/word1/word2/g|在第n1行和n2行之间寻找word1这个字符串，并且将其替换为word2.|
|:1,$s/word1/word2/g：|从第一行到最后一行寻找word1这个字符串，并且将其替换为word2.|
|:1,$s/word1/word2/gc|从第一行到最后一行寻找word1这个字符串，并且将其替换为word2|


###**vim的其他功能**

**1.块选择**

* v:字符选择，会将光标经过的地方反白选择；

* V:行选择；

* Ctrl+v：块选择；

* y：复制反白的地方；

* d：删除反白的地方。

![这里写图片描述](http://img.blog.csdn.net/20170703200401683?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.多文件编辑**

使用命令vim name1 name2 name3...(各个文件名之间用空格隔开)可以同时打开多个文件。

* :n：编辑下一个文件；

* :N：编辑上一个文件；

* :files：列出目前vim打开的所有文件。

![这里写图片描述](http://img.blog.csdn.net/20170703200538111?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**2.多窗口编辑**

输入命令:sp{filename}便可以在一个窗口中打开多个文件。

* Ctrl+w+j/↓ ：光标可以移到下方的窗口；

* Ctrl+w+k/↑ ：光标可以移到上方的窗口；

* Ctrl+w+q：离开

![这里写图片描述](http://img.blog.csdn.net/20170703200346332?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**退出vim编辑器**

**1.在编辑模式下输入**

* w ：将编辑的数据写入到硬盘中。
* q ：退出vim，不保存。
* q! ：强制退出vim ，不保存。
* wq ：保存后退出vim。
* wq! ：强制保存后退出vim


**2.在命令模式下**

使用shift+zz退出并保存


<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！