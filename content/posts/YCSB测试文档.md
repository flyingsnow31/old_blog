---
title: "YCSB测试文档"
date: 2023-02-04T20:08:22+08:00
tags: ['YCSB']
categories: 软件安装与配置
---

# YCSB测试文档

## 测试工具

使用YCSB进行测试

### 简介

Yahoo! Cloud Serving Benchmark (YCSB) 。

Yahoo公司的一个用来对云服务进行基础测试的工具。

使用Java语言实现，如果需要对一个数据库进行性能测试

需要通过java编写客户端代码

YCSB自带了很多主流数据库的客户端代码，可以直接使用

[https://github.com/brianfrankcooper/YCSB](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fbrianfrankcooper%2FYCSB)

### 安装使用

#### 编译环境

需要Maven 3, java >= 8

#### 编译

> 由于网络原因，可能需要对maven进行换源
>
> 参考 [(22条消息) Ubuntu20安装maven并切换阿里镜像源_一时半会想不出的博客-CSDN博客_maven ubuntu 换源](https://blog.csdn.net/a1587943743/article/details/122134892)

```shell
git clone https://github.com/brianfrankcooper/YCSB
cd YCSB
# 编译所有数据库客户端
mvn clean package 
# 编译指定数据库客户端。e.g. mongodb
mvn -pl site.ycsb:mongodb-binding -am clean package
```

对于指定数据库如何编译使用，可以在代码clone下来后，进入指定数据库名字的目录内README文件查看

#### 运行

**推荐**使用 shell 脚本执行。

执行文件在 `./bin` 中，共有四个文件，其中执行文件有三种，分别为python2，shell与bat文件。

如使用shell脚本，记得将所有readme文件中的 `./bin/ycsb` 更换为 `./bin/ycsb.sh`

这三个执行文件的逻辑是：解析命令行、设置java环境，加载java-libs，封装成可以执行的java命令，并执行

```shell
# 命令参数
./bin/ycsb.sh [command] [database] [options]
# [command]
# 	load: 载入数据
# 	run: 执行测试
# [database]
#	指定测试的数据库
#	redis, rocksdb...
# [options]
# 	-P file: 指定workload文件
#	-p key=value: 设置ycsb配置项目，会覆盖或增加workload文件的配置
# 	-s: 中间状态打印到stderr中
# 	-threads n: ycsb客户端的并发测试线程数目，默认是1
```

#### Workloads

在 workloads 目录下，提供了workload的模板以及一些默认workload

| workload名 | 具体细节                |
| ---------- | ----------------------- |
| workloada  | 0.5/0.5, reads/writes   |
| workloadb  | 0.95/0.05, reads/writes |
| workloadc  | 1, read                 |
| workloadd  | 0.95/0.05, reads/insert |
| workloade  | 0.95/0.05, scan/insert  |
| workloadf  | 0.5/0.5, reads/insert   |

workload文件具体配置

| 参数项              | 具体含义                                           |
| ------------------- | -------------------------------------------------- |
| recordcount         | YCSB load阶段加载的记录条数                        |
| operationcount      | YCSB run阶段执行的操作总数                         |
| workload            | workload实现类                                     |
| readallfields       | 查询时是否读取记录的所有字段                       |
| readproportion      | 读操作的百分比                                     |
| updateproportion    | 更新操作的百分比                                   |
| scanproportion      | 插入操作的百分比                                   |
| requestdistribution | 请求分布模式，uniform, zipfian, latest三种分布模式 |

#### 执行过程

一般为先 `load` 再 `run `

load 为将所需的数据生成并写入到指定数据库中

run 为具体执行workload中的操作

#### 测试结果

| 参数                      | 含义                          |
| ------------------------- | ----------------------------- |
| RunTime(ms)               | 数据加载所用时间,单位毫秒(ms) |
| Throughput(ops/sec)       | 吞吐量,即ops(每秒操作次数)    |
| Operations                | 操作的总次数                  |
| AverageLatency(us)        | 平均响应延时,单位是微秒(us)   |
| MinLatency(us)            | 最小响应时间,单位是微秒(us)   |
| MaxLatency(us)            | 最大响应时间,单位是微秒(us)   |
| 95thPercentileLatency(us) | 95%的操作延时,单位是微秒(us)  |
| 99thPercentileLatency(us) | 99%的操作延时,单位是微秒(us)  |
| Return=OK                 | 成功返回数                    |

[READ] 开头表示 `READ` 操作执行的具体情况，其他类似

### 参考

[YCSB数据库性能测试工具 - 简书 (jianshu.com)](https://www.jianshu.com/p/66937631bf0b)

## Kvrocks

### 简介

Kvrocks 是基于 RocksDB 之上兼容 Redis 协议的 NoSQL 存储服务，设计目标是提供一个低成本以及大容量的 Redis 服务，作为 Redis 在大数据量场景的互补服务，选择兼容 Redis 协议是因为简单易用且业务迁移成本低。

https://github.com/apache/incubator-kvrocks

### 安装使用

#### 先决条件

```shell
# Ubuntu / Debian
sudo apt update
sudo apt install -y git gcc g++ make cmake autoconf automake libtool python3 libssl-dev
```

#### 编译

```shell
git clone https://github.com/apache/incubator-kvrocks.git
cd incubator-kvrocks
./x.py build # `./x.py build -h` to check more options;
               # especially, `./x.py build --ghproxy` will fetch dependencies via ghproxy.com.
```

#### 运行

配置文件 `kvrocks.conf` 位于软件根目录

```shell
./build/kvrocks -c kvrocks.conf
```

#### 连接服务

```shell
redis-cli -p 6666

127.0.0.1:6666> PING
PONG
```

### 使用YCSB进行测试

由于kvrocks接口兼容redis协议，于是使用ycsb中的redis

#### ycsb编译

```shell
mvn -pl site.ycsb:redis-binding -am clean package
```

#### 运行kvrock

修改 `kvrocks.conf` ，将db文件目录修改到指定地方，这里修改为750上的一块通过pcie连接的ssd

```shell
# The working directory
#
# The DB will be written inside this directory
# Note that you must specify a directory here, not a file name.
dir /mnt/nvme/kvrocks
```

运行

```shell
./build/kvrocks -c kvrocks.conf
```

#### 运行ycsb进行测试

```shell
./bin/ycsb.sh load redis -s -P workloads/workloada -threads 28 -p redis.host=127.0.0.1 -p redis.port=6666
```

#### 报错

```shell
/usr/bin/java  -classpath /home/lzy/ycsb-0.17.0/conf:/home/lzy/ycsb-0.17.0/lib/HdrHistogram-2.1.4.jar:/home/lzy/ycsb-0.17.0/lib/core-0.17.0.jar:/home/lzy/ycsb-0.17.0/lib/htrace-core4-4.1.0-incubating.jar:/home/lzy/ycsb-0.17.0/lib/jackson-core-asl-1.9.4.jar:/home/lzy/ycsb-0.17.0/lib/jackson-mapper-asl-1.9.4.jar:/home/lzy/ycsb-0.17.0/redis-binding/lib/commons-pool2-2.4.2.jar:/home/lzy/ycsb-0.17.0/redis-binding/lib/jedis-2.9.0.jar:/home/lzy/ycsb-0.17.0/redis-binding/lib/redis-binding-0.17.0.jar site.ycsb.Client -load -db site.ycsb.db.RedisClient -s -P workloads/workloada -threads 28 -p redis.host=127.0.0.1 -p redis.port=6666
redis.clients.jedis.exceptions.JedisConnectionException: java.net.SocketTimeoutException: Read timed out
	at redis.clients.util.RedisInputStream.ensureFill(RedisInputStream.java:202)
	at redis.clients.util.RedisInputStream.readByte(RedisInputStream.java:40)
	at redis.clients.jedis.Protocol.process(Protocol.java:151)
	at redis.clients.jedis.Protocol.read(Protocol.java:215)
	at redis.clients.jedis.Connection.readProtocolWithCheckingBroken(Connection.java:340)
	at redis.clients.jedis.Connection.getIntegerReply(Connection.java:265)
	at redis.clients.jedis.Jedis.zadd(Jedis.java:1385)
	at site.ycsb.db.RedisClient.insert(RedisClient.java:141)
	at site.ycsb.DBWrapper.insert(DBWrapper.java:221)
	at site.ycsb.workloads.CoreWorkload.doInsert(CoreWorkload.java:601)
	at site.ycsb.ClientThread.run(ClientThread.java:135)
	at java.base/java.lang.Thread.run(Thread.java:829)
Caused by: java.net.SocketTimeoutException: Read timed out
	at java.base/java.net.SocketInputStream.socketRead0(Native Method)
	at java.base/java.net.SocketInputStream.socketRead(SocketInputStream.java:115)
	at java.base/java.net.SocketInputStream.read(SocketInputStream.java:168)
	at java.base/java.net.SocketInputStream.read(SocketInputStream.java:140)
	at java.base/java.net.SocketInputStream.read(SocketInputStream.java:126)
	at redis.clients.util.RedisInputStream.ensureFill(RedisInputStream.java:196)
	... 11 more
```

#### 问题分析

> 参考
>
> [(22条消息) jedis的Read timed out异常解决_逆光飞翔2018的博客-CSDN博客_jedis read timeout](https://blog.csdn.net/sinat_25311845/article/details/88051215)

```shell
redis.clients.jedis.exceptions.JedisConnectionException: java.net.SocketTimeoutException: Read timed out
```

表面客户端与服务端的socket连接超时，主要原因推测为数据量过大，导致服务端延时较长。

通过查看ycsb的redis客户端源码，默认的客户端延时为2秒，所以当超时后，客户端主动关闭了连接。

```java
public static final String TIMEOUT_PROPERTY = "redis.timeout";
jedis = new Jedis(host, port, Integer.parseInt(redisTimeout));
```

需要修改jedis的timeout参数，通过在执行ycsb时，使用 `-p` 参数，指定 `redis.timeout` 修改成一个比较大的值即可

同时需要修改kvrocks.conf

```shell
# line 26
# Close the connection after a client is idle for N seconds (0 to disable)
timeout 100000

# The number of worker's threads, increase or decrease would affect the performance.
workers 28
```

#### 完整正确的启动流程

> 使用 `numactl -N 0` 

1. 启动kvrocks

```shell
numactl -N 0 ./build/kvrocks -c kvrocks.conf
```

2. 启动ycsb的load

```shell
numactl -N 0 ./bin/ycsb.sh load redis -s -P workloads/workloada -threads 28 -p redis.host=127.0.0.1 -p redis.port=6666 -p redis.timeout=100000
```

3. 执行ycsb的run

```shell
numactl -N 0 ./bin/ycsb.sh run redis -s -P workloads/workloada -threads 28 -p redis.host=127.0.0.1 -p redis.port=6666 -p redis.timeout=100000
```

4. 收集完成执行结果

5. 关闭kvrocks

6. 删除其在硬盘中的数据

```shell
rm -r /mnt/nvme/kvrocks/*
```

### 测试指标

在750机器上进行测试

ycsb使用28线程，kvrocks的worker也使用28，不限制硬盘空间，默认worker数为8+



![Kvrocks 设计与实现](E:\Study\blog\content\posts\YCSB测试文档\v2-4a9bb286d563121767f03c4c1a1f7cfa_720w-1675512715956.jpg)

主要收集**吞吐率**，**平均延时**，**95%延时**和**99%延时**的数据

## redis

### 简介

Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI [C语言](https://baike.baidu.com/item/C语言?fromModule=lemma_inlink)编写、支持网络、可基于内存亦可持久化的日志型、Key-Value[数据库](https://baike.baidu.com/item/数据库/103728?fromModule=lemma_inlink)，并提供多种语言的API。

[https://github.com/redis/redis](https://github.com/redis/redis)

### 安装使用

```shell
# 下载源码
wget https://download.redis.io/redis-stable.tar.gz

# 编译
tar -xzvf redis-stable.tar.gz
cd redis-stable
make
```

#### 运行

编译后的可执行文件位于 `./src` 中

服务端为 `redis-server` ，客户端为 `redis-cli` 

```shell
./src/redis-server &
./src/redis-cli
127.0.0.1>PING
PONG
```

#### 配置

配置文件位于软件根目录下的 `redis.conf` 中

如果需要使用配置文件，使用如下命令启动

```shell
./src/redis-server redis.conf
```

### 使用YCSB进行测试

#### ycsb编译

编译与对kvrocks进行测试的方式相同

```shell
mvn -pl site.ycsb:redis-binding -am clean package
```

#### 运行redis

选择是否启动持久化，以及持久化的存放目录

```shell
# Save the DB to disk.
#
# save <seconds> <changes> [<seconds> <changes> ...]
#
# Redis will save the DB if the given number of seconds elapsed and it
# surpassed the given number of write operations against the DB.
#
# Snapshotting can be completely disabled with a single empty string argument
# as in following example:
#
# save ""
#
# Unless specified otherwise, by default Redis will save the DB:
#   * After 3600 seconds (an hour) if at least 1 change was performed
#   * After 300 seconds (5 minutes) if at least 100 changes were performed
#   * After 60 seconds if at least 10000 changes were performed
#
# You can set these explicitly by uncommenting the following line.
#
# save 3600 1 300 100 60 10000

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /mnt/nvme/redis
```

使用指定配置启动redis

```shell
./src/redis-server redis.conf
```

#### 运行ycsb进行测试

```shell
./bin/ycsb.sh load redis -s -P workloads/workloada -threads 28 -p redis.host=127.0.0.1 -p redis.port=6379

./bin/ycsb.sh run redis -s -P workloads/workloada -threads 28 -p redis.host=127.0.0.1 -p redis.port=6379
```

### 测试指标

测试开启持久化与关闭持久化下的性能

主要收集**吞吐率**，**平均延时**，**95%延时**和**99%延时**的数据

## RocksDB

### 简介

RocksDB是FaceBook起初作为实验性质开发的，旨在充分实现快存上存储数据的服务能力。由Facebook的Dhruba Borthakur于2012年4月创建的LevelDB的分支，最初的目标是提高服务工作负载的性能，最大限度的发挥**闪存和RAM**的高度率读写性能。

Key和value是任意大小的字节流支持原子的读和写。除此外，RocksDB深度支持各种配置，可以在不同的生产环境（纯内存、Flash、hard disks or HDFS）中调优，RocksDB针对多核CPU、高效快速存储（SSD)、I/O bound workload做了优化，支持不同的数据压缩算法、和生产环境debug的完善工具。

https://github.com/facebook/rocksdb

### 安装使用

#### 依赖

- [zlib](http://www.zlib.net/) - a library for data compression.
- [bzip2](http://www.bzip.org/) - a library for data compression.
- [lz4](https://github.com/lz4/lz4) - a library for extremely fast data compression.
- [snappy](http://google.github.io/snappy/) - a library for fast data compression.
- [zstandard](http://www.zstd.net/) - Fast real-time compression algorithm.

gcc >= 7 

#### 编译

```shell
git clone https://github.com/facebook/rocksdb.git
cd rocksdb
make static_lib # will compile librocksdb.a, RocksDB static library. Compiles static library in release mode.

make shared_lib # will compile librocksdb.so, RocksDB shared library. Compiles shared library in release mode.
```

#### 运行

rocksdb是一个嵌入式的数据库，与redis不同，是通过链接库连接到应用上，与应用一同运行。

### 使用YCSB进行测试

由于rocksdb嵌入式的特性，ycsb通过maven提供了rocksdb的下载，在rocksdb的目录中pom.xml文件可以看到

```xml
<dependency>
      <groupId>org.rocksdb</groupId>
      <artifactId>rocksdbjni</artifactId>
      <version>${rocksdb.version}</version>
</dependency>
```

这里会下载rocksdb在maven仓库中最新的版本，一般为6。可以下载下来直接进行测试，可直接跳到**ycsb编译**部分

#### rocksdb java版本

> 参考
>
> [(22条消息) 【测试】自定义配置 RocksDB 进行 YCSB 测试_ywang_wnlo的博客-CSDN博客](https://blog.csdn.net/CoolBoySilverBullet/article/details/122162072)
>
> [Maven添加本地jar包 - 赵羴 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhao-shan/p/16498061.html)

如果需要使用指定的版本，或者使用rocksdb的最新编译版本，首先需要用java对rocksdb进行编译

java >=8

**注意**需要修改rocksdb的makefile

```makefile
238：ROCKSDB_DISABLE_MEMKIND=true
```

这里关闭rocksdb的MEMKIND，否则在运行ycsb时，会报如下错误

```shell
INFO site.ycsb.db.rocksdb.RocksDBClient - RocksDB data dir: /mnt/nvme/rocksdb
2022-10-31 16:24:26:554 0 sec: 0 operations; est completion in 0 second 
Exception in thread "Thread-23" java.lang.UnsatisfiedLinkError: /tmp/librocksdbjni5496689695031159979.so: /tmp/librocksdbjni5496689695031159979.so: undefined symbol: MEMKIND_DAX_KMEM
```

然后编译java版本的rocksdb

```shell
# 安装依赖
sudo apt-get install libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
# 配置环境变量
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"
# 编译release版本
make -j$(nproc) rocksdbjavastatic
```

编译好的文件位于 `rocksdb/java/target/rocksdbjni-7.8.0-linux64.jar`

然后将此jar文件加入到本地的maven仓库中

```shell
mvn install:install-file
  -Dfile=<path-to-file>
  -DgroupId=<group-id>
  -DartifactId=<artifact-id>
  -Dversion=<version>
  -Dpackaging=<packaging>
  -DgeneratePom=true

Where: <path-to-file>  the path to the file to load
       <group-id>      the group that the file should be registered under
       <artifact-id>   the artifact name for the file
       <version>       the version of the file
       <packaging>     the packaging of the file e.g. jar
```

修改ycsb中rocksdb的pom.xml中对应指定的版本等信息

#### ycsb编译

```shell
mvn -pl site.ycsb:rocksdb-binding -am clean package
```

#### 运行ycsb进行测试

```shell
# 装载数据
./bin/ycsb.sh load rocksdb -s -P workloads/workloada -threads 28 -p rocksdb.dir=/mnt/nvme/rocksdb
# 运行
./bin/ycsb.sh run rocksdb -s -P workloads/workloada -threads 28 -p rocksdb.dir=/mnt/nvme/rocksdb
```

#### 配置

`rocksdb.dir` 指定了rocksdb运行时数据存放在硬盘上的位置

`rocksdb.optionsfile` 指定了运行时rocksdb的配置，可选，如果为空，使用ycsb提供的默认配置

### 测试指标

主要收集**吞吐率**，**平均延时**，**95%延时**和**99%延时**的数据

