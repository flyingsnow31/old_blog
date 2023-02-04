---
title: "Redis Benchmark测试文档"
date: 2023-02-04T20:08:33+08:00
draft: true
tags: ['Redis-benchmark']
categories: 软件安装与配置
---

# Redis-benchmark测试文档

## 测试工具

Redis-benchmark

### 简介

在redis源码中自带了一个性能的测试工具，redis-benchmark

可以模拟N个客户端同时向Redis发送M条查询命令的应用场景。

可以对redis进行指定操作，操作数，值大小等等一系列的压力测试

那么对应同样使用redis协议的kvrocks来说，也是可以使用的

### 编译安装

与编译安装redis一致，在编译完redis后，在其 `src` 目录下即可找到redis-benchmark执行文件

```shell
wget https://download.redis.io/redis-stable.tar.gz
tar -xzvf redis-stable.tar.gz
cd redis-stable
make
```

### 运行

redis-benchmark的运行语法如下

```shell
redis-benchmark [option] [option value]
```

| 选项    | 描述                             | 默认值    |
| ------- | -------------------------------- | --------- |
| -h      | 服务器主机名                     | 127.0.0.1 |
| -p      | 服务器端口                       | 6379      |
| -s      | 服务器socket                     |           |
| -c      | 并发连接数                       | 50        |
| -n      | 请求数                           | 10000     |
| -d      | 字节形式指定值大小               | 2         |
| -k      | 1=keep alive 0=reconnect         | 1         |
| -r      | 随机key                          |           |
| -P      | 通过管道                         | 1         |
| -q      | 强制退出redis，仅显示query/sec值 |           |
| --csv   | 以csv格式输出                    |           |
| -l（L） | 循环，永久测试                   |           |
| -t      | 仅运行以逗号分隔的命令           |           |
| -I（i） | Idle模式，打开n个idle连接并等待  |           |

```shell
$ redis-benchmark -n 10000  -q

PING_INLINE: 141043.72 requests per second
PING_BULK: 142857.14 requests per second
SET: 141442.72 requests per second
GET: 145348.83 requests per second
INCR: 137362.64 requests per second
LPUSH: 145348.83 requests per second
LPOP: 146198.83 requests per second
SADD: 146198.83 requests per second
SPOP: 149253.73 requests per second
LPUSH (needed to benchmark LRANGE): 148588.42 requests per second
LRANGE_100 (first 100 elements): 58411.21 requests per second
LRANGE_300 (first 300 elements): 21195.42 requests per second
LRANGE_500 (first 450 elements): 14539.11 requests per second
LRANGE_600 (first 600 elements): 10504.20 requests per second
MSET (10 keys): 93283.58 requests per second
```

## 测试过程

redis与kvrocks的测试过程几乎一致，仅仅将端口修改为指定端口即可

1. 打开redis/kvrocks
2. 运行指定参数的测试 `./redis-benchmark -h 127.0.0.1 -p 6666 -n 32000000 -c 28 -r 32000000 -d 100 --csv -t get,set -P 16`

3. 收集测试结构

**注意** kvrocks每次测试结束后删除本地的文件

## 测试指标

测试开启/关闭redis持久化，不同worker数的kvrocks等一系列情况下的操作

操作数一般指定为 `32000000`

线程数一般指定为 `28`

value大小一般为 `100 / 1000`

一般测试 `get,set`

管道数可多种情况设置

