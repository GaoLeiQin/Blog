
####**Maven常用命令表**

|命令|功能|
|:----|:----|
|mvn compile|编译源代码|
|mvn test-compile|编译测试代码|
|mvn test|运行测试|
|mvn site| 产生site|
|mvn package|打包|
|mvn install a.jar to b|在本地Repository中安装jar|
|mvn clean|清除产生的项目|
|mvn eclipse:eclipse|生成eclipse项目|
|mvn idea:idea|生成idea项目|
|mvn -Dtest package|组合使用goal命令，如只打包不测试|
|mvn test-compile|编译测试的内容|
|mvn jar:jar| 只打jar包|
|mvn test -skipping compile -skipping test-compile|只测试而不编译，也不测试编译|
|mvn eclipse:clean |清除eclipse的一些系统设置|
|mvn dependency:list|查看当前项目已被解析的依赖|
|mvn deploy|上传到私服|
|mvn clean install-U| 强制检查更新，由于快照版本的更新策略(一天更新几次、隔段时间更新一次)存在，如果想强制更新就会用到此命令|
|mvn source:jar|源码打包|
|mvn -version/-v|显示版本信息 |
|mvn -e|显示详细错误 信息|

<br>

####**Maven实例**


**1.创建Maven项目**

（1）创建普通java项目

```
mvn archetype:create -DgroupId=packageName -DartifactId=projectName
```
（2） 创建Maven的Web项目：

```
mvn archetype:create -DgroupId=packageName -DartifactId=webappName
    -DarchetypeArtifactId=maven-archetype-webapp
```

![这里写图片描述](http://img.blog.csdn.net/20170731220711865?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170731220718505?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**2.打包**

![这里写图片描述](http://img.blog.csdn.net/20170731221025796?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170731221055703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**3.生成站点报告**

![这里写图片描述](http://img.blog.csdn.net/20170731225431528?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！

