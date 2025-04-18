# MySQL 知识体系目录

## 📚 索引与数据结构
1. 为什么MySQL用B+树做索引结构？
2. MySQL索引的最左侧前缀匹配原则
3. MySQL三层B+树能存多少数据？
4. MySQL中的回表是什么？
5. MySQL中使用索引一定有效吗？如何排查索引效果？
6. 详细描述MySQL的B+树查询过程

## ⚙️ 索引优化
1. 索引建立注意事项
2. 索引数量是否越多越好？为什么？
3. 使用EXPLAIN进行查询分析
4. MySQL中如何进行SQL调优？

## 📊 数据类型与函数
1. varchar和char的区别
2. count(*), count(1), count(字段名)的区别

## 🔒 事务与锁机制
1. MySQL事务实现原理
2. MVCC机制详解
3. 脏读、不可重复读和幻读
4. 事务隔离级别详解
5. MySQL默认隔离级别及选择原因
6. MySQL中的锁类型
7. 事务二阶段提交
8. 死锁解决方案

## 📝 日志系统
1. binlog, redo log和undo log的作用与区别
2. MySQL日志类型全解析

## 🚀 性能与高可用
1. 深度分页问题解决方案
2. 主从同步机制实现原理
3. 主从同步延迟处理方案


# Redis 知识体系目录

# 🔴 Redis核心数据类型
1. Redis中常见的数据类型有哪些？
2. Redis的Hash是什么？
3. Redis中Zset实现原理是什么？
4. Redis String类型的底层实现是什么？（SDS）

## ⚡ Redis高性能原理
1. Redis为什么这么快？
2. 为什么Redis设计为单线程？6.0版本为何引入多线程？
3. Redis中跳表的实现原理是什么？

## 🔐 Redis数据一致性与锁
1. Redis中如何保证缓存与数据库的数据一致性？
2. Redis中如何实现分布式锁？
3. 为什么锁需要设置唯一值？
4. Redis的Red Lock是什么？你了解吗？
5. Redis实现分布式锁时可能遇到的问题有哪些？

## 💾 Redis持久化机制
1. Redis的持久化机制有哪些？
2. Redis提供两种主要的持久化机制：

## 🔄 Redis复制与集群
1. Redis主从复制的实现原理是什么？
2. Redis集群的实现原理是什么？

## 🛡️ Redis缓存问题与解决方案
1. Redis中的缓存击穿、缓存穿透和缓存雪崩是什么？
2. Redis数据过期后的删除策略是什么？
3. 如何解决Redis中的热点key问题？
4. Redis中的Big Key问题是什么？
