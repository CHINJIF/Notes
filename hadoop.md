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
##### 测试

```
> bin/hadoop dfs -put ../intput in 
> bin/hadoop dfs -ls ./in/*           // hadoop 分布式文件操作命令
```

```
> bin/hadoop jar hadoop-0.20.2-example.jar wordcount in out     //运行一个叫wordcount的作业
```

```
> bin/hadoop dfs -ls ./out
  /user/grid/out/_logs
  /user/grid/out/part-r-00000
```

* 通过浏览器检查jobtracker在结点50030端口，监控jobtracker
   * http://localhost:50030/jobtracker.jsp
   * list: complete jobs/filed jobs/running jobs/local logs
   * job details: file link, running time, etc
* 通过浏览器访问namenode所在结点在50070端口监控集群
   * browse filesystem
   * cluster summary
   * namenode storage
   * logs

## HDFS
#### 物理存储
* 每个服务器里有路径
   * blk—34820934092830
   * blk—34820934092830_1037.meta

#### 设计思想
* 硬件错误是常态性的，需要冗余
* 流式数据方位，支持数据的批量读取，而**非随机读取**，hadoop擅长**数据分析**而**不**是**事务处理**。NOT OLTP, OLAP
* 简单的一致性模型，为了降低系统复杂性，对文件采取一次性写，多次读。文件一经写入，无法修改
* 程序采用“数据就进”的原则分配节点执行

#### 体系结构
* Namenode
   * 记录文件数据块在哪个datanode的位置和副本信息
   * 元数据操作：事务日志
   * 元数据操作：映射文件
* Datanode
   * 一次写入，多次读取，不修改
   * 文件由数据块组成，一般64MB
* 事务日志
* 映像文件
* Secondary Namenode

#### 可靠性
* 冗余副本策略
   * hdfs—site.xml设置复制因子，指定副本数量
* 机架策略
   * 集群放在不同的机架上，连在同一个交换机上，机架内带宽小
   * HDFS的机架感知
   * 本机架一个副本，别的一个副本
* 心跳机制
   * Namenode接收datanode的心跳和块报告
* 安全模式
   * Namenode在启动时会经过一个“安全模式“
* 校验和
   * 文件创立时都有checksum
   * 校验和会存储在一个隐藏文件夹里用于验证
* 回收站
   * 文件不会立马删除，可以快速恢复
* 元数据保护
   * 映像文件 + 事务日志是namenode的核心数据
   * 增加映像文件 + 事务日志的备份
* 快照

### HDFS文件操作
* 命令行方式
* API方式

###### 列出目录
* HDFS里面没有当前目录的概念，只能ls，不能cd
```
> bin/hadoop dfs -ls ./in/
```

###### 上传文件
* HDFS里面没有当前目录的概念，只能ls，不能cd
```
> bin/hadoop dfs -put path target_path
```

###### 复制到本地
```
> bin/hadoop dfs -get source_path output_path
```

###### 删除
```
> bin/hadoop dfs -rmr path
```

###### 查看内容
```
> bin/hadoop dfs -cat path
```

###### 查看管理信息
```
> bin/hadoop dfsadmin -report
```

###### 安全模式
```
> bin/hadoop dfsadmin -safemode enter
```

#### Hadoop管理节点
##### 增加namenode
* 新机器安装好hadoop
* 复制namenode的配置文件复制到该节点
* 修改masters，slaves文件，增加该节点
* hadoop-daemon.sh start datanode/tasktracker
* 运行start-balancer.sh进行负载均衡

## Map Reduce


## HBASE

## Pig
* pig 是hadoop的客户端
* 提供类似于SQL语句的面向数据流语言
* pig latin 可以进行排序，过滤，求和，分组，关联， 自定义函数。是一种面向数据分析处理的轻量级脚本语言
* pig 可以看作是 pig latin 到 map-reduce的映射器

## Hive
* 把SQL语句映射为mapReduce的查询
