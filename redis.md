# Redis 每日一命令

## 概述
Redis 借鉴了 Linux 操作系统对于版本号的命名规则 :
版本号第二位如果是奇数, 则为非稳定版本(例如2.7、2.9、3.1), 如果是偶数, 则为稳定版本(例如2.6、2.8、3.0、3.2), 当前奇数版本就是下一个稳定版本的开发版本, 例如2.9版本是3.0版本的开发版本, 所以我们在生产环境通常选取偶数版本的 Redis. 当前最新版本为5.0.4为稳定版本. 


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
> PING // 验证是否连接上服务器 服务器连接成功会pong
> SELECT dbindex // 选择数据库 默认16个数据库 0-15 默认进入 0 这些都可以通过 CONFIG 配置
> QUIT // 关闭当前客户端
```

## 服务器命令

INFO 命令

```
> INFO // 会显示服务器统计信息 很实用的一个命令
> MONITOR // 实时打印服务器接收的命令 调试用 例如 : 调试Larave队列
> COMMAND // 获取命令详情数组
> COMMAND COUNT // 获取命令总数
> TIME // 返回当前服务器时间
> DBSIZE // 返回当前数据库的 key 的数量
> FLUSHDB // 删除当前数据库的所有key 谨慎使用
> FLUSHALL // 删除所有数据库的所有的key 更谨慎使用
> CLIENT LIST // 获取连接到服务器的客户端连接列表
```

