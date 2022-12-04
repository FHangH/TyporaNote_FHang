# Hadoop集群2022.11.11 



+   [一、知识点回顾](#_2)
+   [二、启动集群并测试](#_5)
+   [2022.11.15](#20221115_46)

+   [一、安装hadoop出现的问题](#hadoop_47)
+   [二、Hadoop的生态系统](#Hadoop_61)
+   [三、分布式文件系统HDFS](#HDFS_85)

+   [二、HDFS简介](#HDFS_179)
+   [三.HDFS相关概念](#HDFS_191)
+   [四.HDFS体系结构](#HDFS_215)
+   [五.HDFS存储原理](#HDFS_248)
+   [六.HDFS数据读写过程](#HDFS_266)
+   [七.HFS编程](#HFS_278)
+   [](#_298)

# 一、知识点回顾



# 二、启动集群并测试

先删除集群  
rm -rf /usr/local/hadoop-2.7.7/tmp

1、启动集群 start-all.sh  
jps：查看HDFS相关的节点进程  
2720 DataNode  
2897 SecondaryNameNode  
2578  
3491 Jps  
3352 NodeManager  
3066 ResourceManager

HDES相关的节点进程：主从结构，一主多从  
NameNode：名称节点，HDFs的主节点，Master,负责HDFs分布式文件系统元数据的存储与处理  
DataNode：数据节点，HDFS的从节点，通常有1到多个，负责存储数据  
SecondaryNameNode：第二名称节点，辅助NameNode完成HDFS元数据的管理

YARN相关的节点进程：主从结构，一主多从  
ResourceManager：资源管理节点，YARN的主节点，负责整个集群资源的调度与管理  
NodeManager：YARN的从节点，通常有1到多个，负责管理它所在节点的资源

2、测试HDS  
查看：  
用命令 ：haddop fs：查看完整用法  
（HDFS：分布式文件系统）  
Web UI:URI地址，主机名或IP:端口号，默认端口50070  
上传一个大数据文件进行测试  
\[bg@bgserver ~\]$ hadoop fs -mkdir /zhan  
\[bg@bgserver ~\]$ hadoop fs -put VMware-workstation-full-16.0.0-16894299.exe /zhan  
\[bg@bgserver ~\]$ hadoop fs -ls /zhan  
![在这里插入图片描述](https://img-blog.csdnimg.cn/7078e85a16b440fc8fdb49b334e05efa.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/01709f191dda463e8db1f0a5d23ede7d.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/5de9505c4ef44c27bd45fbfe1ce93a5e.png)  
3、测式YARN和MR  
查看：web UI：URI地址，主机名或IP:端口号，默认端口8088  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2346da5984634112b211eb44e103d4b7.png)  
跑应用程序  
![在这里插入图片描述](https://img-blog.csdnimg.cn/93ab2834174b4dab97ae1053d06c4b39.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/216a4ecc9a574eb8b4256b57e5875a1e.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/257411ca8d6a406f91574b8cf95c831d.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3fe64fd068d4001991fffe9b4c0cc47.png)

# 2022.11.15

## 一、安装hadoop出现的问题

1、  
命令未找到：HADOOP HOME环境变量设置  
文件不存在：关注JDR的设置与路径  
2、  
少节点：查看节点日志文件  
userlogs：日志文件  
cat /usr/local/hadoop  
3、  
INFO:信息  
WARN：警告  
ERROR：错误  
FATAL：致命错误  
4、排查：配置文件中结构问题、属性名称、属性值、路径对不对、相关资源权限

## 二、Hadoop的生态系统

一、Hadoop生态系统相关组件  
1、hdfs是Hadoop的分布式文件系统，存储，整个生态系统的基础  
2、YARN，通用的资源管理器  
3、MapReduce,并行计算框架，适用于批处理计算  
4、HBase,基于Hadoop分布式数据库，列式数据库  
5、Hive,基于Hadoop构建的数据仓库，数据存储在HDFS上，可将数据分析的任务写成的SQL查询转换成MapReduce程序进行执行  
6、Spark，大数据内存计算框架，计算速度快  
7、Zookeeper,分布式协调器，为集群运算提供分布式锁等机制  
8、Sqoop，ETL工具，可以实现关系数据库与HDFS之间的数据迁移  
9、Flume，日志收集工具  
10、Storm，流计算框架  
![在这里插入图片描述](https://img-blog.csdnimg.cn/f8dc161524fb499984ca5c51fdca2a28.png)

备注：  
数据仓库和数据库的区别：  
1、数据库存储的是原始数据，没经过任何加工；而数据仓库是为了满足数据分析需要设计的，对源数据进行了ETL（提取转换加载）过程，数据抽取工作分抽取、清洗、转换、装载；  
2、数据仓库的数据量要比数据库大很多。

OLTP 和OLAP区别  
OLTP(Online transaction processing):在线/联机事务处理。典型的OLTP类操作都比较简单，主要是对数据库中的数据进行增删改查，操作主体一般是产品的用户。  
OLAP(Online analytical processing):指联机分析处理。通过分析数据库中的数据来得出一些结论性的东西  
![在这里插入图片描述](https://img-blog.csdnimg.cn/21f414da83f147e6a31f84eb80c9c938.png)

## 三、分布式文件系统HDFS

一、分布式文件系统  
1·分布式文件系统部署由多个节点构成的计算机集群上。  
2.分布式文件系统中的节点通常分类两类：一类称主节点，也称Namenode(名称节点)，用于存储和管理文件系统的元数据；另一类是从节点，也称Datanode(数据节点)，负责存储数据，响应客户瑞的数据读写请求，在主节点的调度下实现数据复制与备份。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/15b58e9899204de9a4b92d584cd86358.png)  
客户端：shell，java API  
![在这里插入图片描述](https://img-blog.csdnimg.cn/35bbe2cf01ee46e294cc5954f8d381b0.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed587bbbdb6e42e0832ee299e2b225cd.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/63a5ded94d3b47238e3091720c4ace63.png)  
连接拒绝  
![在这里插入图片描述](https://img-blog.csdnimg.cn/b853c29671b944b2b2482490c2dda29f.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/c49cdf0704d44dbb852ee6fa88ce0de4.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a7777794f3f4303be8eb5c8bf4eac65.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/3425e0c323c54f34b99ad543b1a9a6d3.png)  
二、HDFS实现的目标  
优点：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/dffa2c87a8464f5881cfaf988a22d732.png)  
不足：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/61fbf3896ae7437da0b378be4d923951.png)  
（3）不支持多用户写入，随机写  
文件的修改； 不适合多次写入，一次读取（少量读取）  
4、不支持多用户的并行写。

![在这里插入图片描述](https://img-blog.csdnimg.cn/ae26bb5ae3f940968f0458f67031a3eb.png)  
2022.11.18  
实训  
1.启动hadoop， 通过jps命令查看启动进程，通过web方式查看namenode和jobtracker，localhost:50070（将查看结果截图放入下方）  
答：  
启动hadoop：start-all.sh  
查看进程:jps  
namenode是HDES Web UI  
jobtracker是YARN Web UI  
HDES Web UI:主机名或IP:50070  
YARN Web UI:主机名或IP:18088  
![在这里插入图片描述](https://img-blog.csdnimg.cn/57af5890fe864ab298a493b3a8c8a73f.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/72718c9addd74e61ab5a51fad6aafeaf.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/01bd649c8d864b68b5ea52929ac4450d.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/ebf89a63dc1245b2b0417352d3ef8114.png)  
小贴士：  
hadoop fs -ls /  
hdfs dfs -ls /  
![在这里插入图片描述](https://img-blog.csdnimg.cn/f6d2c6fe3d114dd59f560cd66291716e.png)  
考试必考  
![在这里插入图片描述](https://img-blog.csdnimg.cn/91ad2f819aae402498d1bfdab5dcfc0e.png)  
1.\[-cat\[-ignoreCrc\]】查看  
cp \[-f\]\[-p I-p\[topax\]\]…\]复杂  
\[-get \[-p\]\[-ignoreCrc\]\[-crc\]…\]  
【-help\[cmd …\]帮助  
\[-mkdir\[-p\]】创建  
\[-mv.】移动  
\[-put\[\[p\]\[】.】上传  
错误原因：磁盘错误  
安全模式：  
一般是off  
![在这里插入图片描述](https://img-blog.csdnimg.cn/e75c9858cba94045a09bedba1de4ba28.png)  
查看安全模式  
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e4ec7217d52472997fb213f27067164.jpeg)  
查看日志：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/a14c3cdf1aa44b25ab9cd169d77ad165.jpeg)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/fc74ea3100b743238a42c6b45596df36.jpeg)

2.  在hdfs根目录下创建tmp目录，通过ls命令查看hadoop下tmp目录中的文件  
    答：  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/90784f1c39ac4e988239e809cb72af96.png)  
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/1991b7dd560445998985c8a2ca0aeef3.png)  
    3.进入hadoop目录，执行命令：hadoop fs -ls / 列出HDFS上的文件  
    答：  
    cd $HADOOP\_HOME

![在这里插入图片描述](https://img-blog.csdnimg.cn/62655121a31342ecbda789b0260c53a7.png)  
4\. 在HDFS上 /user下创建一个你自己拼音名字的目录  
![在这里插入图片描述](https://img-blog.csdnimg.cn/a2db11e682884120a04a1acd6a4c3b45.png)  
5\. 退出hadoop目录，回到用户主目录，创建一个test目录，进入test目录，执行echo “hello world” >test1.txt 命令创建test1.txt文件，并输入hello world内容。  
\>:重定向符

![在这里插入图片描述](https://img-blog.csdnimg.cn/fdfef2e5e52e49969010ca12bc5bef02.png)

6.继续回到hadoop目录下，将刚才在本地创建的test1.txt上传（put）到HDFS下你名字的目录下。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/8147fbb963754be79486e7115d665273.png)  
7\. 查看HDFS下刚才上传的文件的内容  
![在这里插入图片描述](https://img-blog.csdnimg.cn/f15a5e6d16d44233bc58e21ccd93df71.png)  
8.退出hadoop目录，回到用户主目录，彻底删除test目录。  
cd  
rm -rf test  
ll  
![在这里插入图片描述](https://img-blog.csdnimg.cn/baf42752122748df9e7ee5af6902438d.png)  
9.回到hadoop目录下，将HDFS上的test1.txt文件下载到用户主目录并查看  
![在这里插入图片描述](https://img-blog.csdnimg.cn/327e5a169fa5495d9b6afab036108f3c.png)  
hadoop fs -get /user/zhangsan/test1.txt 这个是回到当前目录  
hadoop fs -get /user/hanyang/test1.txt /home/bg 回到主目录  
![在这里插入图片描述](https://img-blog.csdnimg.cn/cae94d7813fd4954b3a03d8c59771a23.png)  
10.在HDFS上删除test1.txt文件。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/e215a568458547749645709cceec1187.png)

# 二、HDFS简介

1.HDFS是谷歌GFS分布式文件系统的开源实现，Hadoop:生态系统分布式文件系统。  
2.HDFS的实现目标：（优点）  
a、兼容廉价的计算机硬件  
b、流数据读写  
c、简单的数据访问模型（一次写入，多次读取）  
d、处理大数据集，文件数据量级在GB到TB级  
3.HDFS的局限性（不足）  
a、不适合低延迟数据访问  
b、无法高效存储大量小文件  
c、不支特多用户写和随机写

# 三.HDFS相关概念

1.块  
（1）HDFS采用文件分块冗余存储的机制，来解决大数据的数据存储与可用性  
（2）HDFS块通常要普通文件系统的块大很多  
（3）问答题 分块存储策略带来的好处：支持大规模文件存储、简化系统设计、适合数据备份  
（4）HDFS与其他分布式文件显著的区别在于：启用机架感知机制  
2.名称节点  
（1）名称节点是HDFS的主节点，存诸HDFS文件系统的元数据  
（2）两个重要数据结构：FsImage和Editlog  
FsImage存放某个时间点上，HDFS文件系统元数据快照  
Editlog存储是对HDFS 文件系统的更新操作  
3.第二名称节点SecondaryNameNode，**辅助节点**

随着时间推移，Editlog会不断变大。HDFS会定期检查Editlog，让SecondaryNameNode启动复制合并机制：SecondaryNameNode会将FsImage和Editlog从NameNode所在节点拉取到自己所在的节点（之后的HDFS更新操作会写入一个新的Editlog文件)，对FsImage和Editlog文件进行合并，得到新的FsImage,再将新FsImage送回Namenode所在节点。

如果Namenode挂掉，存在的元数据丢失，如何恢复集群，SecondaryNameNode有一个FsImage备份（冷备份）  
SecondaryNameNode能起到冷备份作用。

4.数据节点  
（1）数据节点，是HDFS的从节点，通常有多个  
（2）数据节点负责存放数据，负责响应客户端数据的读写请求  
（3）数据节点会定期向名称节点报告自己的信息（心跳信息），说明自己是否可用、数据块列表等  
datanode通过心跳信息告诉namenode自己还在后，namenode才调度读写任务给datanode

# 四.HDFS体系结构

1、概述  
（1）HDFS采用了主从(Master/Slave)结构模型，一个HDFS集群包括一个名称节点(NameNode)和若干个数据节点(DataNode)。  
默认块大小 dfs.blocksize 默认 128MB  
默认块副本数 dfs.replication 默认为3

（2）名称节点作为中心服务器，负责管理文件系统的命名空间及客户端对文件的访问。  
（3）集群中的数据节点一般是一个节点运行一个数据节点进程，负责处理文件系统客户端的读/写请求，在名称节点的统  
一调度下进行数据块的创建、删除和复制等操作。  
（4）每个数据节点的数据实际上是保存在本地Linux文件系统中的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/db2cf45a887944bb8aae03b5251051a0.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/569b715dd7c44de28bee8564f479ecfc.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/a1e7b8cc34fd40f9a98cdeeab9c5abc3.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc2efe263a444876bbb5d2d95e6f2857.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/9858102c5b9a4ae4a8c3f4754d650367.png)

2、HDFS命名空间  
（1）HDFS命名空间：独立、唯一  
（2）HDFS命名空间面向所有应用、项目，相互没有隔离  
3、通信协议  
（1）HDFS采用 TCP/IP协议  
（2）客户端与集群之间、集群内部各节点之间，都是通过RPC （远程过程调用）进行通信  
4、客户端  
（1）HDFS shell 命令： -ls -mkdir -cat -put -get -rm -mv  
（2）HDFS Java API

5、HDFS不足  
（1）命名空间限制  
（2）性能的瓶颈：单点（NameNode）  
（3）隔离问题  
（4）集群可用性，单点故障（SPOF），生产环境可采用Hadoop HA（高可用）、Hadoop联邦机制

# 五.HDFS存储原理

1、数据的冗余存储  
（1）HDFS采用多副本冗余存储策略，默认存储3份，  
（2）多副本存储带来的好处：加快数据传输速度、容易检查数据错误、保证数据的可用性  
2、数据存取策略  
1.数据存放策略  
（1）第1副本：如果是集群内提交，则直接存在本地节点；如果集群外提交，则选择磁盘不太满、CPU不太忙的节点进行存储  
第2个副本：选择与第1个副本不同机架上的节点进行存储，目的是安全（防止交换机出问题）  
第3个副本：选择与第1个副本相同机架上的节点进行存储，目的是数据的高效和可用  
如果有更多副本，则随机选择节点进行存储

2.数据读取策略  
HDFS提供API可用确定数据所在机架 ID，客户端可以调用API来确定自己的机架ID，采用就近原则来读取数据。如果没有，则随机选择一块。

3、数据错误与恢复  
（1）名称节点出错：可采用SecondaryNamenode中的冷备份进行恢复  
（2）数据节点出错：名称节点若没有收到数据节点心跳信息，可将该数据节点标记不可用。若因为数据节点不可用造成文件数据块副本数低于副本数要求，则启动复制机制复制文件块，使副本数达到要求。  
（3）数据出错：HDFS在存储数据时，会计算校验码，存放在数据块文件相同的路径下，读取时会通过校验码对数据进行校验。

# 六.HDFS数据读写过程

1.写数据的过程  
（1）建立FileSystem对象  
（2）创建文件：FileSystem对象create方法，返回FSDataOutputStream对象  
（3）使用FSDataOutputStream对象的write方法写入用户数据  
（4）关闭对象将数据存放HDFS文件系统  
2.读数据的过程  
（1）建立FileSystem对象  
（2）打开文件：FileSystem对象open方法，返回FSDataInputStream对象  
（3）FSDataInputStream对象read方法（借助BufferedReader的readline）来实现数据读取  
（4）关闭对象

# 七.HFS编程

1.HDFS shell  
hadoop fs  
hdfs dfs

2.HDFS-Java API:(导包方式、Maven）  
（1)准备开发环境：JDK+IDEA  
（2)准备Hadoop程序包：  
$HADOOP HOME/share/hadoop目录common、hdfs、mapreduce、yarn四个子目录下所有.jar文件  
![在这里插入图片描述](https://img-blog.csdnimg.cn/60ba60ef73ef4516b3d0145c10a28d5a.png)  
3、启动IDEA，建立Java空项目  
4)在项目引入Hadoop程序包：File–>Project Structure–>Libraries–>±->Java–>找到jar所在目录  
5)新建Java类，编写程序  
Configuration : core-default.xml core-site.xml  
fs.defaultFS RPC  
6)定义包：File->Project Structure->Artifacts->±>JAR->选择带依赖打包->定义main class

7)生成JAR包：Build–>Build Artifacts–>build  
![在这里插入图片描述](https://img-blog.csdnimg.cn/c96ed67f2bd34877838d65f3ad91f954.png)  
2022.11.25

# 

请编写程序，实现如下逻辑：（程序与运行结果均以截图形式提交）

1）从命令行参数读入文件名file1

2.  访问HDFS，判断file1是否存在，若存在则输出“文件已存在”，程序结束

3）若file1不存在，则在HDFS上创建该文件，并写入如下两行信息，其中班级、学号、姓名使用自己的个人信息

```auto
 软件工程xxx班学生名单

 123456789,张三
```

4）成功写入后，程序输出如下提示信息：

HDFS数据写入成功！写入内容如下：

```auto
 软件工程xxx班学生名单

 123456789,张三
```

5）在程序中读从HDFS中读取file1文件内容，并对文件内容进行解析，并参考如下格式输出信息:

大家好！我是张三，我的学号:123456789，我来自软件工程xxx班。

程序整体输出参考下图：  
![在这里插入图片描述](https://img-blog.csdnimg.cn/58f7281b3b274dd7b18d87cc16816dec.png)作业逻辑  
1、读命令行参数  
2、访问HDFS,判断file1是否存在  
3、在HDFS上创建该文件，写入数据  
（写入如下两行信息，其中班级、学号、姓名使用自己的个人信息）  
4、读HDFS文件内容

![在这里插入图片描述](https://img-blog.csdnimg.cn/2ebc38406fb049b5b3e24daeb761424d.png)调用hadoop，fs -ls / 命令行参数

```java
/*
1、读命令行参数
2、访问HDFS,判断file1是否存在
3、在HDFS上创建该文件，写入数据
（写入如下两行信息，其中班级、学号、姓名使用自己的个人信息）
4、读HDFS文件内容
 */
package cn.edu.ahdy.bg;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

import java.io.IOException;

public class ex01 {
    public static void main(String[] args) throws IOException {
                           //hadoop fs -ls /
                                  //fs对应args[0] ls对应args[1] /对应args[2]
        //访问HDFS
        //1.创建Configuration对象
        Configuration conf = new Configuration();
        Path file1 = new Path(args[0]);


        //2.获取FileSystem对象
        FileSystem fs= FileSystem.get(conf);

        //3.创建文件
        FSDataOutputStream out = fs.create(file1);

        //4.写数据
       String str="软件工程212,1234546789，张三";
       out.write(str.getBytes());


       //5.写完数据关闭对象，释放资源
        out.close();
        fs.close();


    }
}

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/103b10b2a37d4a648579567d89c0baa4.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/dd12759831204bdf9cb25b817bb24fbd.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/0b79d4d9c0e9401184716abfac7fd3af.png)![在这里插入图片描述](https://img-blog.csdnimg.cn/6370993b73be4dcb9efb847b861f1d05.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/821208b5e0ee4eadbd232fd05b60d643.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/c80c4bff514a455096dbca80f8e250d9.png)找到jar并导入jar  
![在这里插入图片描述](https://img-blog.csdnimg.cn/4e04fa3189c44e8dbf92a794393a560d.png)查看jps，并启动集群  
![在这里插入图片描述](https://img-blog.csdnimg.cn/ce51c48ba6a3449d8826825e91728b30.png)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/04c0b3bc6988471eab4fb58f153d0de2.png)

```java
/*
1、读命令行参数
2、访问HDFS,判断file1是否存在
3、在HDFS上创建该文件，写入数据
（写入如下两行信息，其中班级、学号、姓名使用自己的个人信息）
4、读HDFS文件内容
 */
package cn.edu.ahdy.bg;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

import java.io.BufferedReader;
import java.io.FilterInputStream;
import java.io.IOException;
import java.io.InputStreamReader;

public class ex01 {
    public static void main(String[] args) throws IOException {
                           //hadoop fs -ls /
                                  //fs对应args[0] ls对应args[1] /对应args[2]
        //访问HDFS
        //1.创建Configuration对象
        Configuration conf = new Configuration();
        Path file1 = new Path(args[0]);


        //2.获取FileSystem对象
        FileSystem fs= FileSystem.get(conf);

        //3.创建文件
        FSDataOutputStream out = fs.create(file1);

        //4.写数据
       String str="软件工程212,1234546789，张三";
       out.write(str.getBytes());


       //5.写完数据关闭对象，释放资源
        out.close();

        //读数据
        //1.打开文件
        FSDataInputStream in = fs.open(file1);
        //2.
        BufferedReader reader= new BufferedReader(new InputStreamReader(in));

        String temp =reader.readLine();
        String[] infos = temp.split(",");
        System.out.printf("大家好！我是%s，我的学号:%s，我来自%s班",infos[2],infos[1],infos[0]);

        reader.close();
        in.close();
        fs.close();

    }
}

```

不指定类![在这里插入图片描述](https://img-blog.csdnimg.cn/79107da416cc4617b0e0b45ad2e1d7bd.png)  
类不存在  
![在这里插入图片描述](https://img-blog.csdnimg.cn/9222e89fa8244ed1aec137f0ca615d78.png)解决方法  
1.

![在这里插入图片描述](https://img-blog.csdnimg.cn/df6df418d6044b489eba903193c2671b.png)2.  
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a4e055b8e9148dea64956884d54097c.png)