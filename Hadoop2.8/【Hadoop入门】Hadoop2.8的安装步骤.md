###**声明**

1.本篇博文介绍的Hadoop（单节点）安装步骤基于以下环境进行

* 虚拟机：VMware Workstation 12.1.0
* 镜像文件：ubuntu16.10

2.Hadoop的安装步骤一共分为5大步，接下来我会一一介绍

###**1.关闭防火墙**

* 安装UFW防火墙：sudo apt-get install ufw

* 查看防火墙状态：sudo ufw status

* 关闭防火墙：sudo ufw disable 

* 默认关闭：sudo ufw default deny 

* 重启Linux：sudo reboot

![这里写图片描述](http://img.blog.csdn.net/20170728111432100?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**2.安装JDK8**

* 安装JDK8：sudo apt-get install sun-java8-jdk

* 配置Java与Hadoop环境变量：sudo vim /etc/profile  之后在文件末尾添加

```
export JAVA_HOME=/home/qingaolei/hadoop/jdk1.8.0_1
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export HADOOP_HOME=/home/qingaolei/hadoop/hadoop-2.8.0
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

```

![这里写图片描述](http://img.blog.csdn.net/20170728125842103?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 使配置文件生效：source /etc/profile

* 验证JDK是否安装成功：java –version

![这里写图片描述](http://img.blog.csdn.net/20170728130003766?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**3.配置SSH免密登录**

* 安装SSH：sudo apt-get install ssh

* 配置无密码登录本机：ssh-keygen -t rsa -P "" 

注：ssh-keygen表示生成密匙，-t表示生成的密匙类型，-p表示生成文件的路径，该命令执行后，会生成两个文件，id_dsa私匙和id_dsa_pub公匙

* 将公钥拷贝到要免登陆的机器上：ssh-copy-id localhost

* 验证SSH是否安装成功：ssh –version

* 验证是否可以免密登录：ssh localhost （第一次登录会询问是否继续连接，输入yes即可）

![这里写图片描述](http://img.blog.csdn.net/20170728112910625?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![这里写图片描述](http://img.blog.csdn.net/20170728112928739?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170728112957181?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**4.安装运行Hadoop（单节点）**

* 下载Hadoop的安装包：
wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz

![这里写图片描述](http://img.blog.csdn.net/20170728114624851?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* 解压安装包：tar –zxvf hadoop-2.8.0.tar.gz

* 修改配置文件：hadoop-env.sh、core-site.xml、hdfs-site.xml、mapred-site.xml、yarn-site.xml

1>修改 hadoop-env.sh 

```
export JAVA_HOME=/home/hzq/software/jdk1.8.0_131  
```

2>修改core-site.xml
```
<configuration>   
        <property>  
                <name>fs.defaultFS</name>  
                <value>hdfs://Ubuntu:9000/</value>  
        </property>  
        <property>  
                <name>hadoop.tmp.dir</name>  
                <value>/home/dream/hadoop/data</value>  
        </property>  
</configuration>
```
注：    fs.defaultFS表示NameNode URI
          hadoop.tmp.dir 表示临时文件的存放地址，默认是"/tmp/hadoop-${user.name}" 

3>修改hdfs-site.xml

```
<configuration>  
        <property>  
            <name>dfs.replication</name>  
            <value>2</value>  
        </property>  
        <property>  
                <name>dfs.block.size</name>  
                <value>64M</value>  
        </property>  
</configuration>  
```
注： dfs.replication 表示Block副本的数量  默认是3
          dfs.block.size表示Block的大小  默认是128M

4>修改mapred-site.xml

```
<configuration>  
        <property>  
                <name>mapreduce.framework.name</name>  
                <value>yarn</value>  
        </property>  
</configuration>
```
注：mapreduce.framework.name 表示制定MR框架为Yarn方式，默认是local

5>修改yarn-site.xml

```
<configuration>  
        <property>  
                <name>yarn.resourcemanager.hostname</name>  
                <value>centos71</value>  
        </property>  
  
        <property>  
                <name>yarn.nodemanager.aux-services</name>  
                <value>mapreduce_shuffle</value>  
        </property>  
<configuration> 
```
注： yarn.resourcemanager.hostname 表示指定Resourcemanager的地址
 yarn.nodemanager.aux-services 表示reduce获取数据的方式


* 格式化namenode（初始化）：hdfs namenode -format

* 启动Hadoop：bin/start-all.sh
 或者先启动HDFS：sbin/start-dfs.sh，之后再启动YARN ：sbin/start-yarn.sh

* 验证Hadoop是否安装成功：输入网址http://localhost:50070	
或者使用jps命令验证

![这里写图片描述](http://img.blog.csdn.net/20170728124559555?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170728124748101?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYmFpeWVfeGluZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

注：
①hdfs namenode –format 命令只是初始化了namenode的工作目录，而DataNode的工作目录是在DataNode运行后产生。
②namenode初始化后形成两个标识：blockpollid和clusterId，新的DataNode加入时会获取这两个ID
③配置文件时，给NameNode配置多个工作目录，可以增强容错性。


<br>
<br>
本人才疏学浅，若有错，请指出，谢谢！ 
如果你在安装过程中遇到什么问题，可以在下方留言，我们一起解决！ 
衷心的感谢您能耐心的读完本篇博文！
