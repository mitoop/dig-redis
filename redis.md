# Redis 每日一命令

## 概述
Redis 借鉴了 Linux 操作系统对于版本号的命名规则 :
版本号第二位如果是奇数, 则为非稳定版本(例如2.7、2.9、3.1), 如果是偶数, 则为稳定版本(例如2.6、2.8、3.0、3.2), 当前奇数版本就是下一个稳定版本的开发版本, 例如2.9版本是3.0版本的开发版本, 所以我们在生产环境通常选取偶数版本的 Redis. 当前最新版本为5.0.4为稳定版本. 

更多细节 :  [Redis 命令参考](http://redisdoc.com/index.html)


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

