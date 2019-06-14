###**引入**

虽然说git命令有很多，但我们平时用的也就不到二十个命令，所以说git也不难掌握，下面我就来介绍下常用的命令。

###**git命令**

####**1.git结构图**


![这里写图片描述](http://img.blog.csdn.net/20170731201746982?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：Workspace：工作区，Index / Stage：暂存区，Repository：仓库区（或本地仓库），Remote：远程仓库

####**2.安装后配置git**

（1）安装 Git 之后，首先需要配置你的名字和邮箱，因为每一次提交都需要这些信息，如下图

![这里写图片描述](http://img.blog.csdn.net/20170731202302843?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）获取git配置信息，查看是否配置成功。

![这里写图片描述](http://img.blog.csdn.net/20170731202434192?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**3.创建版本库**

版本库又名仓库（repository），简单理解成一个目录，这个目录里面的所有文件都可以被Git管理起来，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

（1）初始化版本库

![这里写图片描述](http://img.blog.csdn.net/20170731203210033?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）添加文件到版本库（暂存区）

![这里写图片描述](http://img.blog.csdn.net/20170731203452708?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（3）将文件从暂存区提交到仓库中

![这里写图片描述](http://img.blog.csdn.net/20170731203624422?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：git commit命令的-m参数表示本次提交的说明，可以输入任意内容，便于从历史记录找到改动记录。

（4）当然，一次可以add多个不同的文件，以空格分隔：

![这里写图片描述](http://img.blog.csdn.net/20170731204048089?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


####**4.仓库状态**

（1）显示仓库的状态

![这里写图片描述](http://img.blog.csdn.net/20170731205400069?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：我修改了文件，但并没有commit，所以它给我提示了

（2）查看文件的修改历史

![这里写图片描述](http://img.blog.csdn.net/20170731205257167?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)




####**5.版本回退**

（1）用git log命令查看当前分支的版本历史

![这里写图片描述](http://img.blog.csdn.net/20170731205643606?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）重置暂存区与工作区，与上一次commit保持一致

![这里写图片描述](http://img.blog.csdn.net/20170731205932372?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**6.工作区**

（1）删除工作区文件，并且将这次删除放入暂存区

![这里写图片描述](http://img.blog.csdn.net/20170731210623585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

（2）丢弃工作区的修改（恢复暂存区的指定文件到工作区）

![这里写图片描述](http://img.blog.csdn.net/20170731210319072?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


###**回顾整个过程**

![这里写图片描述](http://img.blog.csdn.net/20170731201635877?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！

> 参考链接：
> 1.[常用 Git 命令清单—阮一峰](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)
> 2.[Git常用命令，很全很详细讲解的也不错](http://blog.csdn.net/afei__/article/details/51476529)
> 3.[Git 常用命令大全](http://blog.csdn.net/dengsilinming/article/details/8000622)