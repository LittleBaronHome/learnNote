# Redis

> 引用官网介绍开篇：
>
> Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes with radius queries and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster

Redis是一个开源的(BSD许可)，内存中的数据结构存储，用作**数据库**，**缓存**和**消息代理**。它支持数据结构，如字符串、散列、列表、集合、使用范围查询的排序集合、位图、hyperloglogs、使用半径查询的地理空间索引和流。Redis有内置的复制，Lua脚本，LRU驱逐，事务和不同级别的磁盘持久性，并提供高可用性通过Redis哨兵和自动分区与Redis集群



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
​	`docker pull redis:5.0.9`

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



## 配置文件

*配置文件对大小写不敏感*

### 引入外部的配置文件

```bash
################################## INCLUDES ###################################
# ...
# include /path/to/local.conf
# include /path/to/other.conf
```

### 加载模块

```bash
################################## MODULES #####################################
# ...
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so
```

### 网络

```bash
################################## NETWORK #####################################
# ...
bind 127.0.0.1 # 默认之只有本机可以连接。可以注释掉，或者改成指定IP
protected-mode yes # 默认yes，开启保护模式，限制为本地访问
port 6379 # 默认端口号
```

### 通用配置
```bash
################################# GENERAL #####################################
# ...
daemonize no # 默认no，改为yes意为以守护进程方式启动，可后台运行
pidfile /var/run/redis_6379.pid # 如果以后台方式启动，则需要指定一个pid文件

# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably)
# warning (only very important / critical messages are logged)
loglevel notice # 日志级别
logfile "" # 默认日志文件位置
databases 16 # 默认数据库数量
```

### 快照(RDB)

> 在规定时间内执行了多少次操作，则会持久化到文件 .rdb .aof
>
> Redis是内存操作，没有持久化就会随着服务关闭丢失数据

```bash
################################ SNAPSHOTTING  ################################
# ...
save 900 1 # 如果900s内，如果至少有1个key进行了改变，则进行持久化操作
save 300 10 # 如果300s内，如果至少有10个key进行了改变，则进行持久化操作
save 60 10000 # 如果60s内，如果至少有10000个key进行了改变，则进行持久化操作

stop-writes-on-bgsave-error yes # 如果持久化出错，继续工作
rdbcompression yes # 是否压缩rdb文件，会消耗cpu资源
rdbchecksum yes # 错误校验rdb文件
dir ./ # rdb文件目录
dbfilename dump.rdb # 默认持久化文件名
```

### 安全

```bash
################################## SECURITY ###################################
# ...
# requirepass foobared # 默认没有密码 可自行设置 requirepass 123456
# 也可通过命令设置 config set requirepass "123456" / config get requirepass
# 连接上cli之后，通过auth 123456命令验证通过之后即可操作
```
### 客户端

```bash
################################### CLIENTS ####################################
# ...
# maxclients 10000 # 做大客户端连接数
```

### 内存

```bash
############################## MEMORY MANAGEMENT ################################
# ...
# maxmemory <bytes> # 最大内存
# maxmemory-policy noeviction # 内存达最大值的处理策略
```

> 六种策略：
>
> * volatile-lru: 只对设置了过期时间的key进行LRU（默认值）
> * allkeys-lru: 删除lru算法的key
> * volatile-random: 随即删除即将过期的key
> * allkeys-random: 随即删除
> * volatile-ttl: 删除即将过期的key
> * noeviction: 返回错误

### AOF

> 持久化

```bash
############################## APPEND ONLY MODE ###############################
# ...
appendonly no # 默认不开启，大部分情况下rdb就够用了
appendfilename "appendonly.aof" # 默认持久化文件名

# appendfsync always # 每次修改就会同步
appendfsync everysec # 每秒同步1次
# appendfsync no # 不同步

```



## redis-benchmark

**redis官方性能测试工具**

​		**`redis-benchmark [option] [option value]`**

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

### Redis-Key常用命令
```bash
# 查看当前数据库key列表
127.0.0.1:6379> keys *
# 查看当前数据库key是否存在
127.0.0.1:6379> exists name
(integer) 1
# 查看当前数据库key值类型
127.0.0.1:6379> type name
string
# 移除当前数据库key
127.0.0.1:6379> move name
(integer) 1

# 为当前数据库key设置有效期（秒）
127.0.0.1:6379> expire name 10
(integer) 1
# 查看当前数据库key剩余有效期（秒）-1永久有效  -2已过期
127.0.0.1:6379> ttl name
(integer) 10
# 为当前数据库key设置值和过期时间，若key已存在则覆盖其值
127.0.0.1:6379[3]> setex name 30 300
OK
```

### 五种常用数据类型

#### String常用命令

``` bash
############################################# 设置值 #############################################
# 设置值
127.0.0.1:6379> set name he
OK
# 批量设置值
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
# 若key不存在时，创建key，若key存在则创建失败
127.0.0.1:6379> setnx name 900
(integer) 1
# 批量设置值 msetnx
127.0.0.1:6379> msetnx k1 v1 k2 v2 k4 v4 # 这是原子操作，有一个key存在，则所有key创建失败
(integer) 0
# 追加值
127.0.0.1:6379> append name llo
(integer) 5
127.0.0.1:6379> get name
"hello"

############################################# 获取值 #############################################
# 获取值
127.0.0.1:6379> get name
"he"
# 批量设置值
127.0.0.1:6379> mget k1 k2 k3
"v1"
"v2"
"v3"

############################################# 组合命令 #############################################
# 先get再set（组合命令）
127.0.0.1:6379> getset name uuu
(nil)
127.0.0.1:6379> getset name iii
"uuu"
127.0.0.1:6379> getset name ooo
"iii"
127.0.0.1:6379> get name 
"ooo"

############################################# 属性操作 #############################################
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

############################################# 累加值 #############################################
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

#### List常用命令

> List是一个两端打开的空间。可以分别从两端存值，故List可以被设计为栈（左存左取）或者队列（右存左取）

```bash
############################################# 设置值 #############################################
# 从左设置值
127.0.0.1:6379> lpush list one
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lpush list three
(integer) 3
# 从左给指定下标复赋值，覆盖原值。下标不能溢出
127.0.0.1:6379> lset list 2 zero
OK
# 从左给指定元素前后添加值
127.0.0.1:6379> linsert list before thr one
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "one"
2) "thr"
# 从右设置值
127.0.0.1:6379> rpush list zero
(integer) 4

############################################# 获取值 #############################################
# 获取值
127.0.0.1:6379> lrange list 0 0
1) "three"
127.0.0.1:6379> lrange list 0 -1
1) "three"
2) "two"
3) "one"
4) "zero"
# 获取指定索引的值
127.0.0.1:6379> lindex list 0
"two"
# 获取长度
127.0.0.1:6379> llen list
(integer) 2

############################################# 移除值 #############################################
# 从左移除一个值
127.0.0.1:6379> lpop list
"three"
# 从右移除一个值
127.0.0.1:6379> rpop list
"zero"

# 移除指定的值
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "two"
4) "three"
5) "two"
6) "one"
7) "zero"
127.0.0.1:6379> lrem list 1 one # 移除1个one值
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "two"
4) "three"
5) "two"
6) "zero"
127.0.0.1:6379> lrem list 1 two # 移除1个two值
(integer) 1
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "three"
3) "three"
4) "two"
5) "zero"
127.0.0.1:6379> lrem list 2 three # 移除2个three值
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "four"
2) "two"
3) "zero"

# 截取值（修改原List值，只保留被截取的值） 
127.0.0.1:6379> ltrim list 1 2
OK
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "zero"

############################################# 组合命令 #############################################
# 从右移出一个值，放进新List中（组合命令）
127.0.0.1:6379> rpoplpush list newlist
"zero"
127.0.0.1:6379> lrange list 0 -1
1) "two"
127.0.0.1:6379> lrange newlist 0 -1
1) "zero"

```

#### Set常用命令

> Set中的值是无序、不重复的

```bash
############################################# 设置值 #############################################
# 设置值
127.0.0.1:6379> sadd set one
(integer) 1
127.0.0.1:6379> sadd set two
(integer) 1
127.0.0.1:6379> sadd set three
(integer) 1

############################################# 获取值 #############################################
# 获取值
127.0.0.1:6379> smembers set
1) "two"
2) "three"
3) "one"
# 随机获取值
127.0.0.1:6379> srandmember set 
"one"
127.0.0.1:6379> srandmember set 2
1) "one"
2) "three"
# 获取长度
127.0.0.1:6379> scard set
(integer) 3
# 判断值是否存在
127.0.0.1:6379> sismember set one
(integer) 1
127.0.0.1:6379> sismember set five
(integer) 0

############################################# 移除值 #############################################
# 移除指定元素
127.0.0.1:6379> srem set two
(integer) 1
127.0.0.1:6379> smembers set
1) "three"
2) "one"

############################################# 组合命令 #############################################
# 将指定元素移到新Set中
127.0.0.1:6379> smove set newset one
(integer) 1
127.0.0.1:6379> smembers set
1) "three"
127.0.0.1:6379> smembers newset
1) "one"

############################################# 交并差 #############################################
127.0.0.1:6379> smembers set
1) "a"
2) "c"
3) "b"
4) "three"
127.0.0.1:6379> smembers newset
1) "c"
2) "one"
# 差集
127.0.0.1:6379> sdiff set newset
1) "a"
2) "three"
3) "b"
# 交集
127.0.0.1:6379> sinter set newset
1) "c"
# 并集
127.0.0.1:6379> sunion set newset
1) "c"
2) "three"
3) "b"
4) "a"
5) "one"
```

#### Hash常用命令

> Map集合 key-value

```bash
############################################# 设置值 #############################################
# 设置值
127.0.0.1:6379> hset hash k1 v1
(integer) 1
# 批量设置值
127.0.0.1:6379> hmset hash k1 v2 k3 v3
OK
# 自增
# hincrby hash k1 1
# hdecrby hash k1 1
# 若key不存在时，创建key，若key存在则创建失败
# hsetnx hash k2 v2

############################################# 获取值 #############################################
# 获取值
127.0.0.1:6379> hget hash k1
"v1"
# 批量获取值
127.0.0.1:6379> hmget hash k1 k2 k3
1) "v2"
2) (nil)
3) "v3"
# 获取全部值
127.0.0.1:6379> hgetall hash
1) "k1"
2) "v2"
3) "k3"
4) "v3"
# 获取长度
127.0.0.1:6379> hlen hash
(integer) 1
# 判断指定字段是否存在
127.0.0.1:6379> hexists hash k1
(integer) 1
# 获取所有key
127.0.0.1:6379> hkeys hash
1) "k1"
# 获取所有value
127.0.0.1:6379> hvals hash
1) "v2"

############################################# 删除值 #############################################
# 删除值
127.0.0.1:6379> hdel hash k3
(integer) 1

```

#### Zset常用命令

> 有序的Set，在Set的基础上，添加一个值 key sort value 

```bash
############################################# 设置值 #############################################
# 设置值
127.0.0.1:6379> zadd zset 1 one
(integer) 1
# 设置多个值
127.0.0.1:6379> zadd zset 2 two 3 three
(integer) 2

############################################# 获取值 #############################################
# 获取值
127.0.0.1:6379> zrange zset 0 -1
1) "one"
2) "three"
3) "two"
# 获取长度
127.0.0.1:6379> zcard zset
(integer) 4 
# 获取指定区间元素数量
127.0.0.1:6379> zcount zset 1 3
(integer) 3

############################################# 排序 #############################################
# 排序
127.0.0.1:6379> zadd zset 0 0 1 1 2 2 3 3 4 4
(integer) 5
# 正常获取就是正序排列
# zrangebyscore zset min max也可以
127.0.0.1:6379> zrange zset 0 -1
1) "0"
2) "1"
3) "2"
4) "3"
5) "4"
# 倒序
127.0.0.1:6379> zrevrange zset 0 -1
1) "4"
2) "3"
3) "2"
4) "0"

############################################# 移除值 #############################################
# 移除指定元素
127.0.0.1:6379> zrem zset 1
(integer) 1
```

### 三种特殊数据类型

#### Geospatial地理位置

#### Hyperloglog基数统计

#### Bitmap位图



## Redis事务

### 概念

>  Redis事务的本质就是一组命令的集合，一个事务所有命令会被序列化，按顺序执行，具有一次性、顺序性、排他性的属性.
>
>  Redis事务的执行步骤：
> * 开启事务 multi
> * 命令入队
> * 执行事务/取消事务 exec/discard

**Redis事务不保证原子性**

**Redis事务没有隔离的概念**

### 命令

```bash
# 开启事务
127.0.0.1:6379> multi
OK
# 命令入队
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> hset hash p1 v1
QUEUED
127.0.0.1:6379> hset hash p1 v2
QUEUED
# 执行事务
127.0.0.1:6379> exec
1) OK
2) (integer) 1
3) (integer) 0
```

### 异常

**运行时异常（如下标溢出），事务中其他的命令可以正常执行**

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
# hash类型的值应该用hset设置
# 该命令相当于给key为test的值赋值为“p1” 并设置a1秒有效期，因a1不是有效的数值类型，故产生运行时异常
127.0.0.1:6379> set test p1 a1
QUEUED
127.0.0.1:6379> set test p2 a1
QUEUED
# 执行命令
127.0.0.1:6379> exec
1) OK
2) (error) ERR syntax error
3) (error) ERR syntax error
# 第一条命令正常执行
127.0.0.1:6379> keys *
1) "k1"
127.0.0.1:6379> get k1
"v1"
```

**编译时异常（如命令格式错误），事务中所有命令均不会被执行**

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set c1 1
QUEUED
127.0.0.1:6379> set c2 2
QUEUED
# 编译时异常会当场抛出
127.0.0.1:6379> set c
(error) ERR wrong number of arguments for 'set' command
127.0.0.1:6379> set c3 3
QUEUED
# 执行失败
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
```

### 乐观锁

> 乐观锁：执行之前获取一个version，执行过程中不会给数据加锁，更新数据的时候会判断数据的version是否发生改变，若version不匹配则不提交事务

**Redis在开始事务之前可以时候watch命令监控可能会改变的数据，若在事务执行exec时，被监控的数据发生改变，则不执行事务**

```bash
127.0.0.1:6379> set money 1000
OK
# 监控money的值
127.0.0.1:6379> watch money
OK
# 开启事务
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 100
QUEUED
# 执行事务之前通过其他线程修改money的值
# 执行失败
127.0.0.1:6379> exec
(nil)
# 如果事务实行失败，记得释放监视，再获取最新的值重新监视
127.0.0.1:6379> unwatch
OK
```

## Redis持节化

### RDB

>**Redis默认的持久化机制**
>
>在指定时间间隔内将内存中的数据集体写入到磁盘中（快照），恢复时就是将文件直接读取到内存里
>
>redis会有一个子进程进行持久化，先将数据写入临时文件，待整个持久化过程结束，再将整个临时文件替换上次的持久化文件，主进程不进行任何IO操作，确保了主进程的性能，但最后一次持久化的数据可能会丢失，故如果需要大规模数据恢复，且数据完整性要求不高的情况下，RDB比AOF更加高效

持久化默认机制及修改方式详见 *配置文件 - 快照（RDB）*

* save规则满足时，会触发生成持久化规则

* 执行flushall命令，会触发持久化规则

* 退出redis，会触发持久化规则

将rdb文件放在redis启动目录(**可用`config get dir`查看**)，redis启动时就会自动加载rdb文件中的数据！

**RDB保存的文件名dump.rdb**

**优点**

* 适合大规模数据恢复

**缺点**

* 随数据完整性要求不高
* 需要一定时间间隔，如果以外宕机，最后一次修改的数据就丢掉了
* 会占用一部分内存空间

### AOF
