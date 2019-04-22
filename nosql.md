## NOSQL 综述

### 特点
* 分布式集群
* 一个产品一般都很有针对性，有其对应的应由场景

### 核心技术
* 存储引擎
  * 行存储
  * 列存储
  * 内存数据库
* 索引设计
  * b+树
  * 位图
* sql优化器
* 事务管理与并发控制
  * 锁
* 容错和恢复

### 数据存储
#### 行式存储
* 数据文件基本组成：块／页
* 缺点，如果需要按列筛选，每一行所有数据都需要读内存

#### 列式存储
* 按列存在块里
* OLAP

#### 内存数据库
* 数据小
* 全部载入内存
* timesten，altibase，soliddb
* 内存索引可以用哈希，外存用b+树

#### 关系型数据库弱点
* 难分布式，因为要不停的join
* 依赖于强大的服务器，贵，hard to scale
* 难以处理非结构化数据，列都要预先定义好

#### 参考书
nosql 数据库入门

### CAP定理
* consistency -- 一致性 ACID，通常牺牲consistency
* availability -- 可用性
* patition tolerance - 分区容忍性，一部分分区坏掉，还是可以用
* cap 只能满足其二

#### 只能满足其二
* ca: rational db
* cp: mongodb
* ap: dynamo, cassandro

### 分类
* key-value 键值数据库:  dynamo, cassandra, riak, voldemort, redis
  * join 变成一系列key-value查询
  * groupby
* 面向文档的数据库，mongodb
  * 处理非结构化数据厉害
* 面向列的数据库， hbase，google bigtable
* 面向图的数据库， neo4j

### 产品
#### redis
* 支持丰富的数据类型：数组，链表，集合

#### big table
##### example
* 学生表需要存，s(s#, snumber, sdepartment, sa)
* 所有的数据都放放在一个大表里

#### mongodb
* 面向文档，所谓文档其实就是一行，但是每一行列是不固定的，为了和行这个概念区分，叫文档

#### new sql
* 关系数据化nosql
* nosql上面包装一个sql解析器--sql外壳
* voltdb
   * OLTP, SQL, 分布式集群，内存数据库

## memcached
* 键值存储，基于内存
* 可以在内存中做为数据库的缓存，介于应用层和数据库之间
* 可以设置数据过期时间，来解决cache更新

### 特点
* 全内存运转，关机后数据丢失
* lru算法导致数据不可控的丢失
* 应用场景有限，主要作为数据库缓存
* 哈希方式存储
* 简单文本协议进行数据通信
* 只操作字符型数据，只是字节流
* 其他类型数据由应用解释，不支持复杂的数据类型，全部依靠应用程序来解释类型，应用端厚，服务器没有多少东西，不负责序列化及反序列化
* 集群采用一致性散列（哈希）算法

### install
* yum install memcached

### start
* cd /etc/rc.d/init.d
* ./memcached start
* memcached -d -p 11212   // start memcached at port 11212 and run in backend

### telnet 测试
* telnet localhost 11211

```
set a_key 0 10 3
a_value

get a_key

delete a_key

```

#### rubygems
* ruby memcached-client
* could store value for number, string, array, dictionary


#### more memcached nodes
* 多个拷贝，可以启动多个memcached
* 保证一致性 - 一致性哈希

### 一致性哈希
* memcached node i 负责管理 keys 满足： hash（key) % n == i
* 假设一共四个节点，那么key： abc 会落到 hash（"abc") % 4 = 2, 2好节点

* 如果一个节点 i 失效，那么所有cache 该 i管的，给i+1
* 如果增加一个节点，i，i-1分一半的key给它

##### hash function
* crc32 冗余校验算法
* 计算hash值很快，甚至可以在硬件里直接算

##### memcached 一致性



#### 高可用方案
* 首先，假设有4个节点，一个down掉，那么25% cache会失效，会造成数据库访问25%的波动
* 高可用方案通过提供replic 保证如果一个节点failed 还有另一个一样的节点，来接管
* repcached

```
repcached -p 11211 -x localhost -v -d 
```
* 对端口11211监听，刚才memcached在那个端口启用，对那个端口的写会自动写到repcached，如果那个端口有失败，那么repcached立即采取行动
* 在应用里如果发现一个端口down，那么可以程序控制来读repcached里的

###### spark storm 也都是基于内存的存贮


## redis
* 是memcached一种改进
* 键值数据库
* c实现
* 稳定性高

#### 特点
* 内存 + 磁盘的持久化保存
* 具有丰富的数据类型
* 自带master slave复制
* 数据快照

#### 支持的数据类型
* string
* 链表
* 集合
* 有序集合
* 散列表

#### 场景
* 时间线应用，e.g 微博，
* 对数组有频繁的添加和删除

#### install
* yum install redis

#### config
* daemonize yes    // 是不是后台运行
* port 6379
* bind 127.0.0.1
* timeout 0       ／／ i seconds 没有client链接 自动关掉
* dir  数据快照写入的目录
* save sec      决定多少次变更之后，把数据写入磁盘持久化

#### op
```
set a_key 4     // 4 字节数
a_value

get a_key

setex foo 5 3    // 3 sec 后过期
```

#### 储存链表
```
lpush data 3
foo

lpush data 3
bar

lrange data 0 -1   // check linked list from head to tail
```

#### 有向集合
```
zadd sets 1 4
hoge

zadd sets 2 4
fuga

zrange sets 0 -1
```

### master slave 复制
* 在redis-server 的配置文件里把该server 配制成某master ip／port的slave
* slave of localhost

* 在master push的数据，在slave里自动复制

#### start
* redis-server config_file

#### 集群管理
* 哈希一致性算法 来管理redis 集群 节点，类似于memcached 多进程
* shard -- 多redis master
* replic／slave -- 多备份
* redis里面没有中心节点，没有central failure，hadoop 的namenode 就有

### redis 数据类型操作与应用
* redis-cli 命令行客户端

#### 场景1 计数器
* 论坛帖子点击数，点击量会造成大量的数据库读写

```
> redis-cli -p 6379

set visits:1:totals 21389     //more key compose keys, 1 means page 1
set visits:2:totals 13243242

incr visits:342:totals    // increase totals
get visits:342:totals
```

#### 场景2 非结构化数据
hset, hget, hincrby  用于操作哈希表，便于存储非结构化数据，e.g 如果每个user带有的属性不同，用redis非常方便
```
hset users:jdoe name "john doe"
hset users:jdoe email "xxx@cc.com"
hset users:jdoe phone "343342342"

hincrby users:jdoe visits 1    // increse by visits by 1

hget users:jdoe email

hgetall users:jdoe

hkyes user:jdoe

hvalue user:jdoe
```

#### 场景3 集合，用集合保存社交网站圈子数据
* 那些网站，属于一个圈子，circle
```
sadd circle:jdoe:family users:mike
sadd circle:jdoe:family users:pite

sadd circile:jdoe:soccer users:brade

smembers circle:jdoe:family

sinter circle:jdoe:family circle:jdoe:soccer   // 求交集
```

#### query for keys
```
keys h*llo
keys h?llo
keys h[ae]llo
```

#### 场景4 QATH 协议
skip

##### 数据模型设计 一个应用 - for OATH 协议
数据：

* consumer keys   加密的密钥
* consumer secrets        lksdlfkdsj
* request tokens 签名
* access tokens
* nonces   登陆的随机码

服务器代码：
```
hgetall  /consumers/key:lksdlfkdsj     // 验证这个用户
sadd  /nonces/key:lksdlfkdsj/timestamp:20192039238    //验证随机码
hset  /request_tokens/key:lksdlfkdsj  ksloiefsjkl ksjdfiejoj  //给key建立一个token

set /authorizations/request_token:ksloiefsjkl
```

#### 场景5 倒排索引
* 文章分词，去掉stop word之后 做倒排索引 inverted index 
* word, [(document_id, offset，word_count), (document_id， offset, word_count)] 
```
//分词后 文章剩下的关键字有 finance, bloomberg, issue
sinter words:finance words:bloomberg word:issue  // 求所有词出现的交集就好 
```

### redis 持久化
* 快照 (snapshot)：作为备份。设定触发条件，发生则把内存里的数据，写入数据库的snapshot文件，e.g. 写入了>100文件。系统奔溃以后，重启会把快照的东西载入内存。会造成数据丢失, 上一次备份，崩溃之间的数据会丢失 .rdb
* AOF (Append Only File)：所有改写数据操作都会写入日志，崩溃，日志重新运行所有命令 .aof

#### 快照
CONFIG
```
save 300 10  // 每10秒钟 有300次修改
dbfilename dump.rdb    // 快照文件 文件名
```

##### 快照原理
* 生成快照时，当前进程fork出一个子进程，写入rdb文件
* 会受到内存大小的限制，容易造成内存/redis的崩溃
* master/slave 也是借助快照文件传递数据

#### AOF
```
appendonly yes   //enable  AOF
```
* AOF 优先于 RDB
* RDB 性能优于 AOF，不会重复更新一条数据
* AOF rewrite, 合并重复aof文件 去掉重复

#### VM
* 为了解决数据库大小受内存限制
* redis不会将全部数据装入内存，把热value放入内存，冷数据放在磁盘
* 但是因为操作系统本身就会做VM优化，所以这个功能其实有重复，后续版本去掉

##### redis 会有的问题
* 快照会引发的问题
  * 写快照，把数据从load back from 快照，会产生很多io，造成阻塞
  * 解决方法：采用master/slave，永远不要再master上设置快照！ 在slave上设置，那么master不会崩溃
* 主从复制阻塞
  * 解决： skip
  
## Mongodb -- 文档数据库
* 基本概念
* 数据类型和数据模型
* 分布式：主从复制
* 分片
* 管理维护：快照，备份
* 应用案例

#### 基本概念
* C++ 编写， 支持Linux，windows，solaris 
* 主要针对**非结构化数据**，对列无限制
* 全面的索引支持，可以在任意属性上建立索引
* 支持map/reduce
* 高可用

#### 面向文档
* 文档是一个区别于表格，行/列的概念
* 无法固定模式/模型，数据结构持续变化的无法被结构化的一条数据，是一个文档

#### 参考书
* mongodb权威指南
* 深入学习mongodb


## Cassandra -- 键值数据库


