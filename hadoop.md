## hadoop
#### 起源
google搜索引擎
* google集装箱数据中心，每个集装箱里又1160台服务器，标准化。能效比1.25.一般公司在2.0
* 解决问题：
    * 怎么存储大量网页，数据冗余，保证安全
    * 搜索算法，如果要模糊匹配 LIKE xxx, 在SQL中是不能用索引的，而是要全表扫描，慢（用倒排索引）
    * page-rank计算问题

#### 倒排索引
```
word       index
Google    {1:1}, {3:2}      // documentId: offset
```
* 分词 -> 去掉stopword -> 倒排索引

##### 分词
* 一种办法，用字典。 切出一个词，在字典里查看有没有

#### page-rank
* 垃圾里面找黄金
* 页面的重要性如何判断？
    * 点击量 -- google爬虫爬不到
    * 通过被引用数量判断，如果被引用得多就证明该页面重要
    * 被page-rank高的页面指向，该页面重要性更高
* google matrix：

```

1 -> 2
|  x |
3 -> 4

0    0    0  0
1/3  0    0  1
1/3  1/2  1  0
1/3  1/2  0  0

G = aS + (1-a)1/n * U
q=Gq, 求上特征矩阵的特征向量。特征向量q[i]就是rank-score
```
通过不断的map-reduce，来进行矩阵的运算，最后出一个千上万列的matrix的特征向量

#### Google tech key
* GFS
* Map-reduce
* Big table

#### lucene
* Hadoop的源起 -- Lucene
* java 做的全文检索工具
* lucene的微缩版：Nutch

### Hadoop子项目
* HBASE, PIG, HIVE
* MapReduce, HDFS, ZooKeeper
* Core, Avro

### 架构
* JobTracker, TaskTracker
* NameNode, DataNode

##### HDFS
* NameNode - Master
   * HDFS的守护程序
   * 作为分布式文件系统的总控作用
   * 对内存和I/O进行集中管理
   * 是单点，有单点故障的风险
* Secondary NameNode 辅助名称节点
   * 作为NameNode的备份
   * 与NameNode通讯，对NameNode快照
* DataNode - Slave
   * 数据分布式存储

##### MapReduce
* JobTracker - Master
   * 用于处理作业(用户提交代码)的后台程序
   * 决定那些文件参与处理，然后切割task并分配节点
   * 监控task，重启失败的task
   * 每个集群只有唯一的一个JobTracker，位于master节点
* TaskTrakcer - Slave
   * 位于Slave节点和与datanode结合
   * 管理个子节点的task，由jobTracker分配
   * 每个节点自会有一个taskTracker，可以启动多个JVM

#### 安装使用
skip

```
>bin/hadoop dfs -put ../intput in 
>bin/hadoop dfs -ls ./in/*           // hadoop 分布式文件操作命令
```

## HDFS

## Map Reduce

## HBASE

## Pig
* pig 是hadoop的客户端
* 提供类似于SQL语句的面向数据流语言
* pig latin 可以进行排序，过滤，求和，分组，关联， 自定义函数。是一种面向数据分析处理的轻量级脚本语言
* pig 可以看作是 pig latin 到 map-reduce的映射器

## Hive
* 把SQL语句映射为mapReduce的查询
