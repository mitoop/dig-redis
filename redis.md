# Redis 每日一命令

## 概述
Redis 借鉴了 Linux 操作系统对于版本号的命名规则 :
版本号第二位如果是奇数, 则为非稳定版本(例如2.7、2.9、3.1), 如果是偶数, 则为稳定版本(例如2.6、2.8、3.0、3.2), 当前奇数版本就是下一个稳定版本的开发版本, 例如2.9版本是3.0版本的开发版本, 所以我们在生产环境通常选取偶数版本的 Redis. 当前最新版本为5.0.4为稳定版本. 

更多细节 :  [Redis 命令参考](http://redisdoc.com/index.html)

所有命令大小写并不敏感, 但是大写是约定, 类似于 Mysql.

## 配置
Redis 的配置文件位于 Redis 安装目录下, 文件名为 `redis.conf`.
可以通过 `CONFIG` 命令查看或设置配置项.

CONFIG 命令
```
// 获取一项配置
CONFIG GET CONFIG_SETTING_NAME 例如 : CONFIG GET requirepass
// 获取所有配置
CONFIG GET *
// 设置一项配置
CONFIG SET CONFIG_SETTING_NAME CONFIG_SETTING_VALUE 例如 : CONFIG SET requirepass mitoop
// 配置持久化
CONFIG REWRITE 
CONFIG REWRITE 命令对启动 Redis 服务器时所指定的 redis.conf 文件进行改写： 因为 CONFIG_SET 命令可以对服务器的当前配置进行修改， 而修改后的配置可能和 redis.conf 文件中所描述的配置不一样， CONFIG REWRITE 的作用就是通过尽可能少的修改， 将服务器当前所使用的配置记录到 redis.conf 文件中。
// 重置 INFO 命令中的某些统计数据 
CONFIG RESETSTAT // 重置 INFO 命令中的某些统计数据
// 其他
要熟悉常用的配置项
```

## 客户端连接
连接命令
```
// 直接连接
> redis-cli -h 127.0.0.1 -p 6379 -a passwod // 直接登录 应该限制 redis 对外访问 密码也应该复杂 性能太高 尝试频率能达到很高
// 或者
> redis-cli -h 127.0.0.1 -p 6379
> AUTH password // 登录
> PING // 验证是否连接上服务器 服务器连接成功会回复 pong
> SELECT dbindex // 选择数据库 默认16个数据库 0-15 默认进入 0 这些都可以通过 CONFIG 配置
> QUIT // 关闭当前客户端
> CLIENT GETNAME // 获取连接名称 默认没有名称 通过 CLIENT SETNAME 设置
> CLIENT SETNAME name // 设置连接名称 可通过 CLIENT GETNAME 或者 CLIENT LIST 看到
> CLIENT SETNAME "" // 重置为空 必须用""
```

## 服务器命令
INFO 命令
```
> INFO // 会显示服务器统计信息 很实用的一个命令
> MONITOR // 实时打印服务器接收的命令 调试用 例如 : 调试 Larave 队列
> COMMAND // 获取命令详情数组
> COMMAND COUNT // 获取命令总数
> TIME // 返回当前服务器时间
> DBSIZE // 返回当前数据库的 key 的数量
> FLUSHDB // 删除当前数据库的所有 key 谨慎使用
> FLUSHALL // 删除所有数据库的所有的 key 更谨慎使用
> CLIENT LIST // 获取连接到服务器的客户端连接列表
> CLIENT KILL 127.0.0.1:43501 // 客户端的ip和端口号 根据 CLIENT LIST 获得
```

## 自动过期
```
> EXPIRE key seconds // 给给定 key 设置生存时间, 当 key 过期时, 它会自动删除. 单位为秒
> EXPIREAT key timestamp // 给给定 key 设置生存时间 时间是 UNIX 时间戳 EXPIRE + AT
> TTL key // 返回给定 key 的剩余生存时间 TLL = time to live
> PERSIST key // 移除给定 key 的生存时间  key 变为永不过期
> PEXPIRE key milliseconds // 给定 key 设置生存时间 单位为毫秒
> PEXPIREAT key milliseconds-timestamp // 给给定 key 设置生存时间 时间是 UNIX 毫秒时间戳 
> PTTL key // 毫秒为单位返回剩余生存时间
```

## 事务

## 持久化
```
Redis 持久化有两种机制. RDB 和 AOF, RDB 备份数据库, AOF 备份执行命令, AOF 类似于 binlog, 但是属于文本格式.

RDB 和 AOF 是可以同时使用的. RDB 文件小, 快照恢复比较快, AOF 文件大, 备份不容易丢数据(much more durable). RDB 是默认持久化方式.
> SAVE // 执行一个同步保存操作, 将当前 Redis 实例的所有数据库快照以 RDB 文件形式保存到硬盘. 同步操作, 单线程会造成阻塞. 
> BGSAVE // 在后台异步保存数据到硬盘, 通过 fork 复制一份当前进程(也会占用同样的内存). 通过 LASTSAVE 查看是否执行成功. RDB 方式.
> BGREWRITEAOF // 执行一个 AOF 文件重写, 重写会创建一个当前 AOF 文件的体积优化版本. 这个是优化 AOF 文件的 例如 写了100次加一 优化为一条加100
> LASTSAVE // 返回最近一次 Redis 成功将数据保存到磁盘上的时间, UNIX时间戳格式.

自动备份, 在 redis.conf 中 : 
RDB
> save N M // 在 N 秒内, 至少发生 M 次修改, 则进行备份. 
> save 900 1 // 默认配置 15分钟内至少有一个键被更改  or 关系
> save 300 10 // 默认配置 5分钟内至少有10个键被更改 or 关系
> save 60 10000 // 默认配置 1分钟内至少有10000个键被更改 or 关系
> dbfilename dump.rdb // 默认备份文件名
> dir ./ // RDB 和 AOF 文件都会存储到该目录

AOF
appendonly no/yes // 关闭/开启 AOF 默认关闭
appendfilename "appendonly.aof" // 默认文件名 在 dir 目录下
// 同步策略 三选一
appendfsync always // 如其名 always 每个修改命令都会调用 fsync 刷新 AOF 文件 慢, 最安全
appendfsync everysec // 默认项 如其名 每秒钟都调用 fsync 刷新到 AOF 文件 快 相对安全 可能会丢失一秒内的数据
appendfsync no // redis 不主动 fsync,依靠系统刷新 最快, 最不安全
配置文件原文 : 
# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

AOF 文件 和 RDB 文件同时存在, 会优先加载时 AOF 文件, 因为数据更完整.
重启redis : 
> 72351:M 19 Apr 2019 00:04:04.244 # Server initialized
> 72351:M 19 Apr 2019 00:04:04.244 * DB loaded from disk: 0.000 seconds // 加载持久化文件
> 72351:M 19 Apr 2019 00:04:04.244 * Ready to accept connections
```

## 过期策略和内存淘汰策略
```
过期策略 : 当 key 过期了, Redis 如何处理 
过期策略机制 : 被动方式 和 主动方式
Redis keys are expired in two ways: a passive way, and an active way.
A key is passively expired simply when some client tries to access it, and the key is found to be timed out.
Of course this is not enough as there are expired keys that will never be accessed again. These keys should be expired anyway, so periodically Redis tests a few keys at random among keys with an expire set. All the keys that are already expired are deleted from the keyspace.
Specifically this is what Redis does 10 times per second:
Test 20 random keys from the set of keys with an associated expire.
Delete all the keys found expired.
If more than 25% of keys were expired, start again from step 1.

淘汰策略 : Redis 内存不足时, 怎么处理新写入的数据.

相关配置项 : 
maxmemory <bytes> // 最大可用内存
当达到最大内存的时候, Redis 触发内存淘汰策略. 需要注意如果配置了 slave 需要考虑合适的最大内存, 给 slave 留内存.
内存淘汰策略 :
maxmemory-policy noeviction
可用配置
# volatile-lru -> Evict using approximated LRU among the keys with an expire set. // 过期 key 执行 LRU Least Recently Used Frequently 最近最少使用 淘汰最长时间未被使用
# allkeys-lru -> Evict any key using approximated LRU. // 所有key 执行 LRU 
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set. // 过期 key 执行 LFU Least Frequently Used 最近最不常用 淘汰一定时期内被访问次数最少
# allkeys-lfu -> Evict any key using approximated LFU. // 所有key 执行 LFU 
# volatile-random -> Remove a random key among the ones with an expire set. // 过期 key 随机删除
# allkeys-random -> Remove a random key, any key. // 随机删除任何 key 
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL) // 删除马上要过期的key
# noeviction -> Don't evict anything, just return an error on write operations. // 不删除 新建时报错 默认值

LRU LFU TTL 不精确 所以每次多拿几个key 然后从中再选一个
maxmemory-samples 5 // 默认取5个key
低版本 Redis 内存淘汰策略和 5.0.4 不太一样
```

## 主从复制
```
ROLE 命令
返回实例在复制中担任的角色, 这个角色可以使 `master`, `slave`, 或者 `sentinel`. 除了角色之外, 命令还会返回与角色相关的其他信息.
> ROLE

SLAVEOF 命令 
将当前服务器转变为指定服务器的从属服务器
> SALVEOF host port
> SALVEOF NO ONE 关闭复制功能, 转变回主服务器, 原来同步数据集不会被丢弃.
```

## 字符串
```
> SET key value [EX seconds] [PX milliseconds] [NX|XX] // 将字符串值 value 关联到 key NX not exist XX exist
> SETNX key value // 只有键 key 不存在的情况下 将 key 的值设置为 value
> SETEX key seconds value // 将键 key 的值设置为 value 并将键 key 的生存时间设置为 seconds 秒
> PSETEX key milliseconds value // 和 SETEX 类似 时间单位为 毫秒
> GET key // 返回 key 对应的元素
> GETSET key value // 将 key 的值设为 value 并返回键 key 在被设置前的旧值
```
## 哈希

## 列表

## 集合
```
> SADD key member [member...] // 集合添加元素
> SMEMBERS key // 查看集合元素 元素过多时候小心使用
> SISMEMBERS key member // 判断 member 是否是 key 的成员
> SPOP key // 移除并返回集合中的一个随机元素
> SRANDMEMBER key [count] // 随机返回集合中的一个(几个)元素
> SREM key member [member...] // 删除集合元素
> SMOVE source destination member // 将元素从 source 集合移动到 destination 集合
> SCARD key // 查看集合基数
> SINTER key [key...] // 求集合的交集
> SUNION key [key...] // 求集合的并集
> SDIFF key [key...] // 求集合的差集
```

## 有序集合
