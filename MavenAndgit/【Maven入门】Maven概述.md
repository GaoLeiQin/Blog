####**1.定义**

* Maven 作为 Apache 的一个开源项目，旨在给项目管理提供更多的支持，它最早的意图只是为了给 apache 组织的几个项目提供统一的开发、测试、打包和部署，能让开发者在多个项目中方便的切换。

* Maven 中最值得称赞的地方就是使用了标准的目录结构和部署。

* 在多个开发团队环境的情况下，Maven可以设置在上班的路上在很短的时间内为标准。由于大部分的项目设置简单可重复使用，Maven的生活的开发容易，而创建报告，检查，生产和测试的自动化设置。

* maven是一个项目构建和管理的工具，提供了帮助管理 构建、文档、报告、依赖、scms、发布、分发的方法。可以方便的编译代码、进行依赖管理、管理二进制库等等。

* maven的好处在于可以将项目过程规范化、自动化、高效化以及强大的可扩展性，利用maven自身及其插件还可以获得代码检查报告、单元测试覆盖率、实现持续集成等等。

####**2.Maven的基本原理**

* Maven 的基本原理很简单，采用远程仓库和本地仓库以及一个类似 build.xml 的 pom.xml ，将 pom.xml 中定义的 jar 文件从远程仓库下载到本地仓库，各个应用使用同一个本地仓库的 jar ，同一个版本的 jar 只需下载一次，而且避免每个应用都去拷贝 jar 。

* 同时它采用了现在流行的插件体系架构，只保留最小的核心，其余功能都通过插件的形式提供，所以 maven 下载很小，在执行 maven 任务时，才会自动下载需要的插件。 

![这里写图片描述](http://img.blog.csdn.net/20170731212752905?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* mirror相当于一个拦截器，它会拦截maven对remote repository的相关请求，把请求里的remote repository地址，重定向到mirror里配置的地址。

![这里写图片描述](http://img.blog.csdn.net/20170731212949147?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 此时，B Repository被称为A Repository的镜像。如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。换句话说，任何一个可以从仓库Y获得的构件，都能够从它的镜像中获取。

* < mirrors/>是镜像列表，是maven从远程仓库里下载构件的一组服务器镜像。镜像能减轻中央maven库的负载，也能突破代理等的网络环境的限制，每个仓库都有一个ID，而mirror需要和仓库的ID对应。

####**3.坐标**

（1）定义

坐标用来标识时空中的某个点，方便人们找到位置，如中电信息大厦可以用经纬度坐标找到，也可以通过国家、省市区、街道、门牌组成的坐标去找。

（2）分类

* groupId: 组织ID，一般是公司、团体名称
* artifactId：实际项目的ID，一般是项目、模块名称
* version:版本，开发中的版本一般打上 SNAPSHOT 标记
* Type/packaging :包类型，如JAR,EAR,POM...
* classifier:分类，如二进制包、源、文档

通过这个规则就可以定位到世界上任何一个构件。

####**4.特点**


* 依赖管理是maven的一大特征，对于一个简单的项目，对依赖的管理并不是什么困难的事，但是如果这个项目依赖的库文件达到几十个甚至于上百个的时候就不是一个简单的问题了。在这个时候maven对于依赖管理的作用就显露出来了。

* 传递性依赖是在maven2中添加的新特征，这个特征的作用就是你不需要考虑你依赖的库文件所需要依赖的库文件，能够将依赖模块的依赖自动的引入。

* 由于没有限制依赖的数量，如果出现循环依赖的时候会出现问题，这个时候有两种方式处理，一种是通过 build-helper-maven-plugin 插件来规避，另一种就是重构两个相互依赖的项目。

* 通过传递性依赖，项目的依赖结构能够很快生成。Maven 能够解决依赖传递

* 传递依赖中需要关注的就是依赖调解，依赖调解的两大原则是：最短路径优先和第一声明优先

* maven有三套classpath（编译classpath，运行classpath，测试classpath）分别对应构建的三个阶段。依赖范围就是控制依赖与这三套classpath的关系。

* 依赖范围有六种：
 * compile：编译依赖范围，在三个classpath都有效。  
 * test：测试依赖范围，在编译代码和运行代码是无效。  
 * provided：以提供的依赖范围，在编译和测试的时候有效，在运行的时候无效。例如servlet-api,因为容器已经提供，在运行的时候是不需要的。  
 * runtime：运行时依赖范围，仅在测试和运行的时候有效。例如jdbc只有在测试和运行的时候才有效。  
 * system：系统依赖范围，与provided范围一致，但是依赖是通过系统变量来指定依赖，不利于移植。
 * import(在maven2.0.9后支持)：导入依赖范围，对三个classpath没有实际影响。

![这里写图片描述](http://img.blog.csdn.net/20170731213701600?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**5.三级仓库结构**

（1）远程公用仓库

Maven 内置了远程公用仓库： http://repo1.maven.org/maven2 这个公用仓库也叫中央仓库是由 Maven 自己维护，包好了世界上大部分流行的开源项目构件。

（2）内部中央仓库

也称私有共享仓库(私服)。一般是由公司自己设立的，只为本公司内部共享使用。它既可以作为公司内部构件协作和存档，也可以作为公用类库镜像缓存，减少在外部访问和下载的频率。

（3）本地仓库

Maven 会将工程中依赖的构件(Jar包)从远程下载到本机一个目录下管理，通常默认在 $user.home/.m2/repository 下。

![这里写图片描述](http://img.blog.csdn.net/20170731213932908?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170731214010594?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

####**6.maven的生命周期**

（1）clean：在真正的构建之前进行一些清理工作

* pre-clean
* clean
* post-clean

（2）default：构件项目的核心部分，编译、测试、打包、部署等

* validate:验证工程是否正确，所有需要的资源是否可用。   
* compile:编译项目的源代码， 
* test-compile:编译项目测试代码。   
* test:使用已编译的测试代码，测试已编译的源代码。   
* package:已发布的格式，如jar，将已编译的源代码打包。   
* integration-test:在集成测试可以运行的环境中处理和发布包。   
* verify:运行任何检查，验证包是否有效且达到质量标准。   
* install:把包安装在本地的repository中，可以被其他工程作为依赖来使用  
* deploy:在整合或者发布环境下执行，将最终版本的包拷贝到远程的repository，使得其他的开发者或者工程可以共享。   
* generate-sources:产生应用需要的任何额外的源代码，如xdoclet。

（3）site：生成项目报告、站点、发布站点

* pre-site
* site
* post-site
* site-deploy

<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你有更好的建议，可以留言我们一起讨论，共同进步！ 
衷心的感谢您能耐心的读完本篇博文！


