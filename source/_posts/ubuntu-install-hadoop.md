title: Ubuntu14.04下安装Hadoop2.6.0 （单机分布式）
tags: []
categories:
  - hadoop
date: 2015-01-22 22:19:00
category:
---
###一、在Ubuntu下创建hadoop组和hadoop用户
####1、创建hadoop用户组

```bash
sudo addgroup hadoop
```
####2、创建hadoop用户
```bash
sudo adduser -ingroup hadoop hadoop
```
输入密码后一路回车就行了
####3、为hadoop用户添加权限
```bash
sudo vim /etc/sudoers
```
加入
```text
hadoop  ALL=(ALL:ALL) ALL
```

###二、用新增加的hadoop用户登录Ubuntu系统
反正最好是用hadoop账号做后续的事情就是的了

###三、安装ssh
如果没装oepnssh的话，先装上
```bash
sudo apt-get install openssh-server
```
然后
```bash
ssh localhost
```
完事，也可以从别的机器登陆进来

###四、安装JDK
```bash
sudo apt-get install openjdk-7-jdk
```
输入
```bash
java -version
```
查看是否安装成功，成功后应该是如下提示
```text
java version "1.7.0_65"
OpenJDK Runtime Environment (IcedTea 2.5.3) (7u71-2.5.3-0ubuntu0.14.04.1)
OpenJDK 64-Bit Server VM (build 24.65-b04, mixed mode)
```

###五、安装hadoop2.6.0
####1、下载安装
从下载官方的包到tmp目录中，解压后拷贝到/usr/local下
```bash
cd /tmp
wget -c http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
sudo tar xzf hadoop-2.6.0.tar.gz
sudo mv hadoop-2.6.0 /usr/local/hadoop  
```
赋予774权限
```bash
sudo chmod 774 /usr/local/hadoop
```
####2、配置
#####1)配置/etc/environment
先查看一下本地jdk安装路径
```bash
update-alternatives --config java
```
我的显示如下 
```
There is only one alternative in link group java (providing /usr/bin/java): /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java
Nothing to configure.
```
只取前面的部分
```bash
/usr/lib/jvm/java-7-openjdk-amd64
```
配置/etc/environment文件，加入
```bash
JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
```
生效并验证一下
```bash
source /etc/environment
echo $JAVA_HOME
```
#####2)编辑hadoop-env.sh
将/usr/local/hadoop/etc/hadoop/hadoop-env.sh中找到JAVA_HOME变量，修改为
```text
export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
```
#####3)配置core-site.xml

在文件/usr/local/hadoop/etc/hadoop/core-site.xml的`<configuration></configuration>`之间增加如下内容
```xml
<property>
    <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
</property>
```
有些时候在后面执行hadoop命令的时候，会报
`Unable to load native-hadoop library for your platform`

这时候加入以下配置节使用本地库
```xml
<property>
<name>hadoop.native.lib</name>
  <value>true</value>
  <description>Should native hadoop libraries, if present, be used.</description>
</property>
```
#####4)配置yarn-site.xml
在文件/usr/local/hadoop/etc/hadoop/yarn-site.xml的`<configuration></configuration>`之间增加如下内容
```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
```
#####5)创建和配置mapred-site.xml
将/usr/local/hadoop/etc/hadoop/mapred.xml.template重名为mapred.xml 
在该文件的`<configuration></configuration>`之间增加如下内容
```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```
#####6)配置hdfs-site.xml
在文件/usr/local/hadoop/etc/hadoop/配置hdfs-site.xml的`<configuration></configuration>`之间增加如下内容
```xml
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/usr/local/hadoop/hdfs/name</value>
</property>
<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/usr/local/hadoop/hdfs/data</value>
</property>
```
###六、格式化hdfs
进入hadoop文件夹
```bash
cd /usr/local/hadoop
```
格式化hdfs
```bash
bin/hdfs namenode -format
```
###七、启动hadoop
```bash
sbin/start-dfs.sh
```
如果提示yes/no，输入yes即可
然后执行
```bash
sbin/start-yarn.sh
```
观察执行效果，输入jps命令，可以看到hadoop运行情况
```text
1913 SecondaryNameNode
2162 NodeManager
3145 Jps
1750 DataNode
2058 ResourceManager
1496 NameNode
```
浏览器打开http://localhost:50070/，会看到hdfs管理页面

浏览器打开http://localhost:8088，会看到hadoop进程管理页面
###八、WordCount验证
####1、dfs上创建input目录
```bash
bin/hadoop fs -mkdir -p input
```
把hadoop目录下的README.txt拷贝到dfs新建的input里
```bash
bin/hadoop fs -copyFromLocal README.txt input
```
运行WordCount
```bash
bin/hadoop jar share/hadoop/mapreduce/sources/hadoop-mapreduce-examples-2.6.0-sources.jar org.apache.hadoop.examples.WordCount input output
```
运行效果
```text
15/01/22 04:46:06 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
15/01/22 04:46:07 INFO input.FileInputFormat: Total input paths to process : 1
15/01/22 04:46:07 INFO mapreduce.JobSubmitter: number of splits:1
15/01/22 04:46:07 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1421930484281_0002
15/01/22 04:46:08 INFO impl.YarnClientImpl: Submitted application application_1421930484281_0002
15/01/22 04:46:08 INFO mapreduce.Job: The url to track the job: http://ubuntu:8088/proxy/application_1421930484281_0002/
15/01/22 04:46:08 INFO mapreduce.Job: Running job: job_1421930484281_0002
15/01/22 04:46:15 INFO mapreduce.Job: Job job_1421930484281_0002 running in uber mode : false
15/01/22 04:46:15 INFO mapreduce.Job:  map 0% reduce 0%
15/01/22 04:46:20 INFO mapreduce.Job:  map 100% reduce 0%
15/01/22 04:46:27 INFO mapreduce.Job:  map 100% reduce 100%
15/01/22 04:46:27 INFO mapreduce.Job: Job job_1421930484281_0002 completed successfully
15/01/22 04:46:27 INFO mapreduce.Job: Counters: 49
....
```
运行完毕后，查看单词统计结果
```bash
hadoop fs -cat output/*
```
结果如下
```text
(BIS),    1
(ECCN)	1
(TSU)	1
(see	1
5D002.C.1,	1
740.13)	1
<http://www.wassenaar.org/>	1
....
```
至此，hadoop本机的搭建过程到此完毕



###⑨、参考文章
1.[一、Ubuntu14.04下安装Hadoop2.4.0 （单机模式）](http://www.cnblogs.com/kinglau/p/3794433.html)

2.[二、Ubuntu14.04下安装Hadoop2.4.0 （伪分布模式）](http://www.cnblogs.com/kinglau/p/3796164.html)

3.[Unable to load native-hadoop library for your platform Hadoop本地库与系统版本不一致引起的错误解决方法](http://www.360doc.com/content/14/0724/15/597197_396743445.shtml)