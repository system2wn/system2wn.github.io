---
layout: post
title:  hadoop大数据系统构建（一）：hadoop环境配置
date:   2017-08-03 13:37:01 +0800
categories: 代码
tag: 大数据
---

前言
==

	暑假进行了为期三周的实训，实训项目是完成一个大数据系统，可以分析接入系统的app的各项数据，我负责系统后台的工作，也就是大数据分析,从对大数据一窍不通到最终完成系统后台，这三周的工作我打算用博客记录下来，记录自己的学习历程。

在Ubuntu16.4上搭建hadoop 2.7.3
==========================
系统的大数据处理框架使用hadoop，所以开始正式工作前，要配置好开发环境，操作系统使用Ubuntu16.4，hadoop版本使用2.7.3。

一、搭建Java环境
----------
（1）在oracle官网下载jdk-8u131-linux-x64.tar.gz并解压：
新建/usr/java目录，然后在shell中切换到压缩包所在目录，然后将压缩包解压到/usr/java目录下：

```
tar -zxvf jdk-8u131-linux-x64.tar.gz -C /usr/java/
```
（2）配置环境变量：
修改.bashrc：

```
sudo vim ~/.bashrc
```

在最后一行写入下列内容：

```
export JAVA_HOME=/usr/java/jdk1.8.0_131
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
export PATH=$JAVA_HOME/bin:$PATH
```
运行如下命令使环境变量生效。

```
source ~/.bashrc
```
 同样的，打开并修改profile文件：

```
 sudo vim /etc/profile
```
在最后一行写入下列内容：
```
export JAVA_HOME=/usr/java/jdk1.8.0_131 
export JAVA_BIN=$JAVA_HOME/bin
export JAVA_LIB=$JAVA_HOME/lib
export CLASSPATH=.:$JAVA_LIB/tools.jar:$JAVA_LIB/dt.jar 
export PATH=$JAVA_HOME/bin:$PATH
```
打开并修改environment 文件：

```
sudo vim /etc/environment
```
追加jdk目录和jdk下的lib的目录到文件中：

```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/java/jdk1.8.0_131/lib:/usr/java/jdk1.8.0_131"  
```
使配置生效：

```
source /etc/environment
```
上述配置全部完成后，java环境配置也应该成功了，下面语句可以验证java环境是否配置成功，如果能够成功显示java的版本，则配置成功：

```
java -version
```

二、安装ssh-server并实现免密码登录
----------------------
（1）下载ssh-server：

```
sudo apt-get install openssh-server
```
（2）启动ssh：

```
sudo /etc/init.d/ssh start
```
（3）查看ssh服务是否启动，如果有显示相关ssh字样则表示成功：

```
ps -ef|grep ssh
```
（4）设置免密码登录：
使用如下命令，一直回车，直到生成了rsa：

```
ssh-keygen -t rsa
```
导入authorized_keys：

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
测试是否免密码登录localhost：

```
ssh localhost
```
关闭防火墙：

```
ufw disable
```

三、伪分布模式安装hadoop
---------------
（1）下载hadoop-2.7.3.tar.gz，并解压到/usr/local：

```
sudo tar zxvf hadoop-2.7.3.tar.gz -C /usr/local
```
切换到/usr/local下，将hadoop-2.7.3重命名为hadoop，并给/usr/local/hadoop设置访问权限：

```
cd /usr/local
sudo mv hadoop-2.7.3 hadoop 
sudo chmod 777 /usr/local/hadoop
```
（2）配置.bashrc文件：

```
sudo vim ~/.bashrc
```
在文件末尾追加下面内容，然后保存：

```
#HADOOP VARIABLES START 
export JAVA_HOME=/usr/java/jdk1.8.0_131 
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL 
export HADOOP_COMMON_HOME=$HADOOP_INSTALL 
export HADOOP_HDFS_HOME=$HADOOP_INSTALL 
export YARN_HOME=$HADOOP_INSTALL 
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
#HADOOP VARIABLES END 
```
执行下面命令，使添加的环境变量生效：

```
source ~/.bashrc
```
（3）hadoop配置 (伪分布模式搭建)：
配置hadoop-env.sh：

```
sudo vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

```
# The java implementation to use. 
export JAVA_HOME=/usr/java/jdk1.8.0_131 
export HADOOP=/usr/local/hadoop
export PATH=$PATH:/usr/local/hadoop/bin
```
配置yarn-env.sh：
```
# export JAVA_HOME=/home/y/libexec/jdk1.6.0/ 
JAVA_HOME=/usr/java/jdk1.8.0_131
```
配置core-site.xml：
```
<configuration>
 <property>
	<name>fs.default.name</name>
	<value>hdfs://localhost:9000</value>
 </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/hadoop_tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    
    <property>  
        <name>hadoop.tmp.dir</name>  
        <value>/usr/local/hadoop/filesystem/tmp</value>  
        <description> temporary directories.</description>  
    </property>  

    <property>  
        <name>dfs.name.dir</name>  
        <value>/usr/local/hadoop/filesystem/name</value>
        <description>where on the local filesystem the DFS name node should store the name table</description>  
    </property>  

    <property>  
        <name>dfs.data.dir</name>  
        <value>/usr/local/hadoop/filesystem/data</value>  
        <description>where on the local filesystem an DFS data node should store its blocks.</description>  
    </property>  
</configuration>
```
根据上面的配置，需要创建/usr/local/hadoop/hadoop_tmp，/usr/local/hadoop/filesystem/tmp，/usr/local/hadoop/filesystem/name，/usr/local/hadoop/filesystem/data这四个目录。

配置hdfs-site.xml：

```
<configuration>
        <property>
             <name>dfs.replication</name>
             <value>1</value>
        </property>
        <property>
             <name>dfs.namenode.name.dir</name>
             <value>file:/usr/local/hadoop/hadoop_tmp/dfs/name</value>
        </property>
        <property>
             <name>dfs.datanode.data.dir</name>
             <value>file:/usr/local/hadoop/hadoop_tmp/dfs/data</value>
        </property>

    <property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
    </property>

</configuration>
```
配置mapred-site.xml：

```
<configuration>
 <property>
	<name>mapred.job.tracker</name>
	<value>localhost:9001</value>
 </property>
</configuration>
```
（4）关机重启系统，使配置生效。

四、测试Hadoop是否安装并配置成功
--------------------
（1）验证Hadoop单机模式安装完成：

```
hadoop version
```
若成功显示hadoop版本，则说明单机配置成功。
（2）伪分布式启动hdfs：
格式化namenode：

```
hdfs namenode -format
```
有 "……has been successfully formatted" 等字样出现即说明格式化成功。

> 注意：每次格式化都会生成一个namenode对应的ID，多次格式化之后，如果不改变datanode对应的ID号，运行wordcount向input中上传文件时会失败。

启动hdfs：
```
start-all.sh
```
然后显示进程：

```
jps 
```
如果NameNode、DataNode和SecondaryNameNode同时都存在，则说明伪分布式hadoop配置并启动成功。
在浏览器中输入http://localhost:50070/，出现如下页面：
<img src="{{ '/styles/images/2017-08-03-hadoop大数据系统构建（一）：hadoop环境配置/50070.png' | prepend: site.baseurl }}" alt="50070"/>
输入 http://localhost:8088/，出现如下页面：
<img src="{{ '/styles/images/2017-08-03-hadoop大数据系统构建（一）：hadoop环境配置/8088.png' | prepend: site.baseurl }}" alt="8088"/>

则说明伪分布安装配置成功了。

停止伪分布式hdfs：

```
stop-all.sh
```

五、总结
----

	至此，hadoop开发环境的搭建已经完成，下一篇博客将记录如何在hadoop上运行案例，以及eclipse上集成hadoop开发环境的配置以及案例的运行。