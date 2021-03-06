# redis 特点

1. 速度快
2. 丰富的数据类型
3. 特性丰富

# redis 用途

1. 缓存
2. 分布式锁

# redis 数据结构

1. string
2. hash
3. list
4. set
5. zset

# redis 主从同步

## 主从同步的流程

首先全量同步，然后增量同步。全量同步时，首先 fork 出一个子进程，并生成 .rdb 文件，然后将其发送到从节点，从节点应用此 .rdb 文件来初始化数据。然后就进入增量同步状态，redis 主节点将命令备份，并将其发送到从节点执行。

## 主从同步时怎么应付网络抖动

redis 在同步时会使用一个复制挤压缓冲区来备份已经发送的数据，当 slave 请求同步时，redis 会使用这个复制挤压缓冲区来判断是否可以进行增量同步，如果能，就从这个复制挤压缓冲区冲复制命令到从节点执行。如果不能，就执行全量复制。

# redis 集群

## 集群的扩容机制

## hash tag

# 使用细节

## 到达内存上限时，redis 的淘汰策略

当数据集上升到一定大小时，就会施行数据淘汰策略。

1. no-enviction：禁止淘汰数据，这时内存不够时，会直接返回错误。
2. volatie-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰。
3. volatie-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰。
4. volatie-random：从已设置过期时间的数据集中选择任意的数据淘汰。
5. allkeys-lru：从数据集中挑选最近最少使用的数据淘汰。
6. allkeys-random：从数据集中选择任意数据淘汰。

> 设置 maxmemory-samples 可以配置淘汰时挑选的样本数

## 用 redis 充当缓存

1. key 的过期机制：定时扫描或者在获取 key 的时候检查 key 的时间戳，折中的策略是抽取 + 即时检查。
2. 缓存击穿：redis 集群中不存在，从而导致大量的访问请求直接压向数据库。
3. 缓存雪崩：设置随机的过期时间来避免大量的数据同时过期。
4. 淘汰机制：LRU
5. 缓存一致性：先更新数据库，再删除缓存。

## redis 实现分布式锁

- 获得锁：获得锁可以通过 set 命令来执行，为了避免获得锁的节点因为崩溃而导致死锁，可以通过设置一个超时时间来到期之后自动释放锁。除了使用 set 命令之外，还可以通过脚本来定制锁。
- 释放锁：直接删除 key 即可，或者等待 key 超时而被释放。

## redis 实现消息队列

用 list 可以实现轻量级消息队列。lpush 将消息加入队列，lpop 将消息从队列中取出。但是安全性不足，当 redis 崩溃后，数据会丢失。

## 怎么避免 keys 命令带来的隐患

在使用 keys 命令时，redis 会遍历所有的 key 找到匹配的数据，如果此时的数据量很大，就会长时间阻塞。为了避免这种情况，可以使用 scan 命令来代替 keys 命令，scan 以迭代的方式遍历 redis 中的数据，并返回一个用于后续迭代的游标，这样后续的 scan 指令就接着上一次的扫描位置继续向后扫描。
