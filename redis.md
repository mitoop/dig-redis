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
CONFIG SET CONFIG_SETTING_NAME CONFIG_SETTING_VALUE 例如 : CONFIG SET requirepass mitoop
// 其他
要熟悉常用的配置项
```



