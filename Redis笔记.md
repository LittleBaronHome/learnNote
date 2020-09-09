# Redis

> 引用官网介绍开篇：
>
> Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes with radius queries and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster

Redis是一个开源的(BSD许可)，**内存**中的数据结构存储，用作**数据库**，**缓存**和**消息代理**。它支持数据结构，如**字符串**、**散列**、**列表**、**集合**、**使用范围查询的排序集合**、**位图**、**hyperloglogs**、**使用半径查询的地理空间索引**和**流**。Redis有内置的复制，Lua脚本，LRU驱逐，事务和不同级别的磁盘持久性，并提供高可用性通过Redis哨兵和自动分区与Redis集群

## 	安装

**本文采用5.0.9版本**

### linux安装

> **Redis官网下教程: https://redis.io/download**

#### 下载安装

```bash
# 下载安装包
cd /opt
wget http://download.redis.io/releases/redis-5.0.9.tar.gz
# 解压
tar xzf redis-5.0.9.tar.gz
cd redis-5.0.9
# 安装
make
```

#### 启动服务

redis默认不是后后台启动， 修改配置文件

> **配置文件路径：/解压路径/redis-5.0.9/redis.conf**
>
> **建议将配置文件复制一份，后续修改启动均使用复制的配置文件，保持原版配置文件不变，便于恢复**
>
> **bind 127.0.0.1** #注释掉这部分，这是限制redis只能本地访问
>
> **protected-mode no** #默认yes，开启保护模式，限制为本地访问
>
> **daemonize no**#默认no，改为yes意为以守护进程方式启动，可后台运行

```bash
# 启动Redis
cd /解压路径/redis-5.0.9/
# 指定配置文件启动
src/redis-server ../redis.conf 
```

#### redis-cli验证

```bash
# 链接redis-cli测试
# 默认端口6379，可在配置文件修改
src/redis-cli -p 6379
127.0.0.1:6379> exit
```

#### 关闭服务

```bash
# 关闭服务
src/redis-cli -p 6379
127.0.0.1:6379> shutdown
not connected> exit
```



### Docker安装

> **DockerHub: https://hub.docker.com/_/redis**

#### 拉取redis镜像
	`docker pull redis:5.0.9`

#### 启动服务

```bash
# 后台启动linux，将/data文件挂载到宿主机/docker/host/dir
# redis-server --appendonly yes 持久化
docker run --name redis -p 6379:6379 -v /usr/local/docker/data:/data -d redis:5.0.9 redis-server --appendonly yes
```

#### 指定配置文件启动

```bash
# 以指定配置文件启动
wget http://download.redis.io/releases/redis-5.0.9.tar.gz
# 解压
tar xzf redis-5.0.9.tar.gz
cd redis-5.0.9
# 复制出配置文件
cp redis.conf /usr/local/docker/

# 按需求修改配置文件

# 指定配置文件启动
docker run --name redis -p 6379:6379 -v /usr/local/docker/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/data:/data -d redis:5.0.9 redis-server /usr/local/docker/redis.conf --appendonly yes
```

#### redis-cli验证

```bash
# 进入容器
docker exec -it redis /bin/bash
# 链接redis-cli
root@6f9ae5d1f5ce:/data# redis-cli
127.0.0.1:6379> exit
root@6f9ae5d1f5ce:/data# 
```

## redis-benchmark

**redis官方性能测试工具**

		**`redis-benchmark [option] [option value]`**

**注意**：该命令是在 redis 的目录下执行的，而不是 redis 客户端的内部指令。

| 序号 | 选项      | 描述                                       | 默认值    |
| :--- | :-------- | :----------------------------------------- | :-------- |
| 1    | **-h**    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**    | 指定服务器端口                             | 6379      |
| 3    | **-s**    | 指定服务器 socket                          |           |
| 4    | **-c**    | 指定并发连接数                             | 50        |
| 5    | **-n**    | 指定请求数                                 | 10000     |
| 6    | **-d**    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**    | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **--csv** | 以 CSV 格式输出                            |           |
| 12   | **-l**    | 生成循环，永久执行测试                     |           |
| 13   | **-t**    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | **-I**    | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

**linux启动在redis-benchmark所在目录`/解压目录/redis-5.0.9/src`下执行**

**docker启动，进入容器直接执行**

```bash
# 测试100000个请求，100个并发量
redis-benchmark -c 100 -n 100000
```

**结果分析**

```bash
# 测试set命令
====== SET ======
# 100000个请求一共用时4.71秒
  100000 requests completed in 4.71 seconds
# 100个并行客户端
  100 parallel clients
# 每次写入3个字节
  3 bytes payload
# 保持一个链接
  keep alive: 1

#每毫秒完成度
0.01% <= 1 milliseconds
49.74% <= 2 milliseconds
86.93% <= 3 milliseconds
93.94% <= 4 milliseconds
96.69% <= 5 milliseconds
97.89% <= 6 milliseconds
98.40% <= 7 milliseconds
98.92% <= 8 milliseconds
99.22% <= 9 milliseconds
99.43% <= 10 milliseconds
99.56% <= 11 milliseconds
99.73% <= 12 milliseconds
99.76% <= 13 milliseconds
99.84% <= 14 milliseconds
99.92% <= 15 milliseconds
99.94% <= 17 milliseconds
99.95% <= 18 milliseconds
99.95% <= 35 milliseconds
99.95% <= 37 milliseconds
99.96% <= 38 milliseconds
99.99% <= 39 milliseconds
100.00% <= 39 milliseconds
21240.44 requests per second
```

## Redis数据库

redis默认有16个数据库（配置文件中可改），默认使用第0个，可通过select index改变

```bash
root@6f9ae5d1f5ce:/data# redis-cli
# 切换数据库
127.0.0.1:6379> select 3 
OK
# 查看当前数据库使用量
127.0.0.1:6379[3]> dbsize
(integer) 0

# 清空当前数据库
127.0.0.1:6379[3]> flushdb
OK
# 清空所有数据库
127.0.0.1:6379[3]> flushall
OK
```



## Redis数据类型

> 官网文档： It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes with radius queries and streams. 

**它支持的数据结构有字符串、散列、列表、集合、使用范围查询的排序集合、位图、基数统计、使用半径查询的地理空间索引和流。**

### Redis-Key

```bash
# 查看当前数据库key列表
127.0.0.1:6379> keys *
# 查看当前数据库key是否存在
127.0.0.1:6379> exists name
(integer) 1
# 移除当前数据库key
127.0.0.1:6379> move name
(integer) 1
# 为当前数据库key设置有效期（秒）
127.0.0.1:6379> expire name 10
(integer) 1
# 查看当前数据库key剩余有效期（秒）
127.0.0.1:6379> ttl name
(integer) 10
# 查看当前数据库key值类型
127.0.0.1:6379> type name
string
```

### String

``` bash
# 设置值
127.0.0.1:6379> set name he
OK
# 获取值
127.0.0.1:6379> get name
"he"
# 追加值
127.0.0.1:6379> append name llo
(integer) 5
127.0.0.1:6379> get name
"hello"

# 获取字符串长度
127.0.0.1:6379> strlen name
(integer) 5
# 截取字符串
127.0.0.1:6379> getrange name 0 3
"hell"
# 替换字符串
127.0.0.1:6379> setrange name 1 xx
(integer) 5
127.0.0.1:6379> get name
"hxxlo"

# 当前值+1
127.0.0.1:6379> set views 0
OK
127.0.0.1:6379>  incr views
(integer) 1
# 当前值-1
127.0.0.1:6379> decr views
(integer) 0
# 当前值+10
127.0.0.1:6379> incrby views 10
(integer) 10
# 当前值-10
127.0.0.1:6379> decrby views 10
(integer) 0
```





