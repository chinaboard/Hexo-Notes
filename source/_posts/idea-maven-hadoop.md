title: IDEA使用maven创建hadoop项目
tags: []
categories:
  - hadoop
date: 2015-01-25 02:14:00
---
有点绕，大概就这么个意思
### 添加依赖
先装好maven，新建一个maven项目，从pom.xml中添加依赖
源：[mvnrepository](http://mvnrepository.com/tags/hadoop)
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-core</artifactId>
        <version>2.6.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-common</artifactId>
        <version>2.6.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-common</artifactId>
        <version>2.6.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
        <version>2.6.0</version>
    </dependency>
</dependencies>
```
### 新建类
这个不用多说了

### 编译Jar包
File-Project Structure或Ctrl+Shift+Alt+S（吐槽一下这什么鸟快捷键欺负HHKB吗）
进入Project Structure，在Artifacts里，添加Jar-Empty
在Output Layout里，添加Module Ouput，选择指定模块就行了
如果模块只包含一个main函数，则自动选择，否则需要手工选择
图省事儿的话还可以选上"Build On make "(make 项目的时候会自动输出Jar)
这样直接Build Project就可以看到Jar包了

### 注意
#### 运行参数问题
默认的hadoop运行参数一般都这样
`hadoop jar [jar包] [input路径] [output路径]`
但是由于我们指定了主类，所以运行参数应变为这样
`hadoop jar [input路径] [output路径]`

#### JDK版本问题
有些时候本地IDE的JDK版本大于Hadoop的运行版本
在IDEA中，调整一下Project bytecode version即可

### 参考文章
1.[Hadoop on Mac with IntelliJ IDEA - 4 制作jar包](http://www.cnblogs.com/michaellfx/p/4001148.html)
2.[IntelliJ IDEA 将 Maven 构建的 Java 项目打包](http://www.cnblogs.com/xuesong/p/3644597.html)
3.[IntelliJ IDEA 打包Source Jar的方法](http://www.hankcs.com/program/java/methods-intellij-idea-source-jar-package.html)
4.[用Maven构建Hadoop项目](http://blog.fens.me/hadoop-maven-eclipse/)
5.[编写简单的Mapreduce程序并部署在Hadoop2.2.0上运行](http://www.tuicool.com/articles/7Jr632)
