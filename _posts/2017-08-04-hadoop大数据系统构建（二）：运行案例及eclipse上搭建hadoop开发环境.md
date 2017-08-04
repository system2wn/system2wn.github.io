---
layout: post
title:  hadoop大数据系统构建（二）：运行案例及eclipse上搭建hadoop开发环境
date:   2017-08-04 23:55:01 +0800
categories: hadoop
tag: 大数据
---
前言
==

	前面一篇已经介绍了系统开发环境的搭建，本篇将记录样例程序WordCount如何在hadoop上运行，以及如何在eclipse上搭建hadoop开发环境，并且在eclipse上运行WoudCount程序。

运行WordCount
==========================
WordCount程序是hadoop的入门程序，类似于java初学者最先学习的helloworld程序，话不多说，看一下WordCount的源码：

{% highlight java %}
public class WordCount {

	public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {

		private final static IntWritable one = new IntWritable(1);
		private Text word = new Text();

		public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
			StringTokenizer itr = new StringTokenizer(value.toString());
			while (itr.hasMoreTokens()) {
				word.set(itr.nextToken());
				context.write(word, one);
			}
		}
	}

	public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
		private IntWritable result = new IntWritable();

		public void reduce(Text key, Iterable<IntWritable> values, Context context)
				throws IOException, InterruptedException {
			int sum = 0;
			for (IntWritable val : values) {
				sum += val.get();
			}
			result.set(sum);
			context.write(key, result);
		}
	}

	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf, "word count");
		job.setJarByClass(WordCount.class);
		job.setMapperClass(TokenizerMapper.class);
		job.setCombinerClass(IntSumReducer.class);
		job.setReducerClass(IntSumReducer.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileInputFormat.addInputPath(job, new Path(args[1]));
		FileOutputFormat.setOutputPath(job, new Path(args[2]));
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}
{% endhighlight %}

这个程序的功能是处理输入文件中的以空格相互隔开的字符串，得到每个字符串的出现个数，并输出到输出文件中。

下面介绍运行步骤：

（1）首先启动hdfs：

```
start-all.sh
```
（2）查看hdfs下包含的文件目录：
```
hadoop dfs -ls /
```
运行后如下图所示：
<img src="{{ '/styles/images/2017-08-04-hadoop大数据系统构建（二）：运行案例及eclipse上搭建hadoop开发环境/dfs_ls.png' | prepend: site.baseurl }}" alt="dfs_ls"/>

如果你是第一次运行hdfs，则什么也不会显示。
（3）在hdfs中创建一个文件目录input：
```
hdfs dfs -mkdir /input
```
运行后应如下图所示：
<img src="{{ '/styles/images/2017-08-04-hadoop大数据系统构建（二）：运行案例及eclipse上搭建hadoop开发环境/dfs_mkdir.png' | prepend: site.baseurl }}" alt="dfs_mkdir"/>

（4）将/usr/local/hadoop/README.txt上传至input中：
```
hadoop fs -put /usr/local/hadoop/README.txt /input
```
（5）执行以下命令运行wordcount，并将结果输出到output中：
```
hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /input /output
```
如果执行成功，在hdfs中会出现output目录。
（6）执行成功后output 目录下会生成两个文件：_SUCCESS 成功标志的文件，里面没有内容。 一个是 part-r-00000 ，通过以下命令查看执行的结果：
```
hadoop fs -cat /output/part-r-00000
```
结果大致如图所示：
<img src="{{ '/styles/images/2017-08-04-hadoop大数据系统构建（二）：运行案例及eclipse上搭建hadoop开发环境/dfs_part-r-00000.png' | prepend: site.baseurl }}" alt="dfs_part-r-00000"/>

至此，WordCount程序的运行便完成，我们也大致知道了hadoop工作的流程，从输入目录中读取文件并做处理，处理后输出到输出文件中，只要写好处理逻辑，便可得到我们想要的结果。

下面提供一些hdfs常用的命令，供以后使用：
```
hadoop fs -mkdir /tmp/input       在HDFS上新建文件夹 
hadoop fs -put input1.txt /tmp/input 把本地文件input1.txt传到HDFS的/tmp/input目录下 
hadoop fs -get input1.txt /tmp/input/input1.txt 把HDFS文件拉到本地 
hadoop fs -ls /tmp/output         列出HDFS的某目录 
hadoop fs -cat /tmp/ouput/output1.txt 查看HDFS上的文件 
hadoop fs -rmr /home/less/hadoop/tmp/output 删除HDFS上的目录 
hadoop dfsadmin -report 查看HDFS状态，比如有哪些datanode，每个datanode的情况 
hadoop dfsadmin -safemode leave 离开安全模式 
hadoop dfsadmin -safemode enter 进入安全模式
```

eclipse上搭建hadoop开发环境
====================

一、安装eclipse
----------
要使用eclipse，首先需要有java环境，java环境的配置在第一篇博客中有介绍。安装好java环境后，在eclipse官网下载eclipse-jee-neon-3-linux-gtk-x86_64.tar.gz并解压到/opt/jvm目录下：
```
sudo tar zxvf eclipse-jee-mars-2-linux-gtk-x86_64.tar.gz -C /opt/jvm
```
然后打开eclipse的图标就可以了。

如果打开后提示没有安装JDK，JRE环境，在/opt/eclipse/文件夹中创建一个指向JRE路径的软链接，bash切换到eclipse所在目录，使用以下命令即可：
```
sudo ln -sf $JRE_HOME jre
```
如果能够成功打开eclipse进入主界面，则eclipse安装完成。

二、eclipse配置hadoop开发环境
----------------------
（1）下载hadoop插件hadoop-eclipse-plugin-2.7.3.jar，并将插件移动到eclipse安装目录下的plugins文件夹下。

（2）重新启动eclispe。

（3）配置hadoop安装目录：打开【Windows】—>【Preferences】后，在窗口左侧会有Hadoop Map/Reduce选项，点击此选项，在窗口右侧设置hadoop安装路径，然后点击【OK】。

（4）配置hdfs端口：打开【Windows】–>【Perspective】–>【Open perspective】–>【Other】，选择【Map/Reduce】，点击【OK】。

（5）点击【Map/Reduce Location】选项卡，点击右边小象图标，打开Hadoop Location配置窗口：

输入Location Name，任意名称即可。配置Map/Reduce Master和DFS Mastrer，Host和Port配置成与core-site.xml的设置一致即可，点击【Finish】。

（6）点击左侧的DFSLocations—>MyHadoop（上一步配置的location name)，如果不报错，表示安装成功。

三、在eclipse中运行WordCount
---------------
（1）新建hadoop项目：
点击【File】—>【Project】，选择【Map/Reduce Project】，输入项目名称WordCount，一直回车。

（2）新建WordCount类，将上面的WordCount源码写入类中，如图：
<img src="{{ '/styles/images/2017-08-04-hadoop大数据系统构建（二）：运行案例及eclipse上搭建hadoop开发环境/wordcount.png' | prepend: site.baseurl }}" alt="wordcount"/>

（3）点击WordCount.java，右键，点击【Run As】—>【Run Configurations】，配置运行参数，即输入和输出文件夹，如图：
<img src="{{ '/styles/images/2017-08-04-hadoop大数据系统构建（二）：运行案例及eclipse上搭建hadoop开发环境/runconfiguration.png' | prepend: site.baseurl }}" alt="runconfiguration"/>

（4）点击【Run】，运行程序，运行结束后，在【DFS Locations】可以查看输出结果，如图：
<img src="{{ '/styles/images/2017-08-04-hadoop大数据系统构建（二）：运行案例及eclipse上搭建hadoop开发环境/dfslocation.png' | prepend: site.baseurl }}" alt="dfslocation"/>

如果最后得到了正确的输出文件，则说明我们的一切开发环境都已配置好，可以开始正式的开发了。

> 第一次在eclipse中运行WordCount程序时，可能会出现控制台打印输出出错的问题，这是因为hadoop中集成了log4j输出框架，需要提供配置文件。
> 
> 解决方法是在项目的src目录下建立一个log4j.propeties文件，里面写入下列内容即可：

```
log4j.rootLogger=debug, stdout, R 
#log4j.rootLogger=stdout, R   
log4j.appender.stdout=org.apache.log4j.ConsoleAppender   
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout   
#log4j.appender.stdout.layout.ConversionPattern=%5p - %m%n   
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n  
log4j.appender.R=org.apache.log4j.RollingFileAppender   
log4j.appender.R.File=log4j.log   
log4j.appender.R.MaxFileSize=100KB   
log4j.appender.R.MaxBackupIndex=1   
log4j.appender.R.layout=org.apache.log4j.PatternLayout   
#log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n   
log4j.appender.R.layout.ConversionPattern=%d %p [%c] - %m%n  
#log4j.logger.com.codefutures=DEBUG
```

总结
==

	至此，所有的开发环境都已经搭建完成，便可以正式进行系统功能的开发，下一篇将记录一些具体功能的实现算法与步骤。