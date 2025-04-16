## 1.Redis中常见的数据类型有那些？
<font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 常见的数据结构主要有五种，这五种类型分别为：String (字符串)、List (列表)、Hash、Set (集合)、Zset (有序集合，也叫 sorted set)。</font>

### <font style="color:rgb(0, 0, 0);">String</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">字符串是 Redis 中最基本的数据类型，可以存储任何类型的数据，包括文本、数字和二进制数据。它的最大长度为 512MB。</font>

#### <font style="color:rgb(0, 0, 0);">使用场景：</font>
+ **<font style="color:rgb(0, 0, 0) !important;">缓存</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：存储临时数据，如用户会话、页面缓存。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">计数器</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：用于统计访问量、点赞数等，通过原子操作增加或减少。</font>

### <font style="color:rgb(0, 0, 0);">Hash</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">哈希是一个键值对集合，适合存储对象的属性。Redis 内部使用哈希表实现，适合小规模数据。</font>

#### <font style="color:rgb(0, 0, 0);">使用场景：</font>
+ **<font style="color:rgb(0, 0, 0) !important;">商品详情</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：存储商品的各个属性，方便快速检索。</font>

### <font style="color:rgb(0, 0, 0);">List</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">列表是有序的字符串集合，支持从两端推入和弹出元素，底层实现为双向链表。</font>

#### <font style="color:rgb(0, 0, 0);">使用场景：</font>
+ **<font style="color:rgb(0, 0, 0) !important;">消息队列</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：用于简单任务调度、消息传递等场景，通过 LPUSH 和 RPOP 操作实现生产者消费者模式。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">历史记录</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：存储用户操作的历史记录，便于快速访问。</font>

### <font style="color:rgb(0, 0, 0);">Set</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">集合是无序且不重复的字符串集合，使用哈希表实现，支持快速查找和去重操作。</font>

### <font style="color:rgb(0, 0, 0) !important;">使用场景：</font>
+ **<font style="color:rgb(0, 0, 0) !important;">标签系统</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：存储用户的兴趣标签，避免重复。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">唯一用户集合</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：记录访问过某个页面的唯一用户，方便进行分析。</font>

### <font style="color:rgb(0, 0, 0) !important;">Sorted Set</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">有序集合类似于集合，但每个元素都有一个分数（score），用于排序。底层使用跳表实现，支持快速的范围查询。</font>

### <font style="color:rgb(0, 0, 0) !important;">使用场景：</font>
+ **<font style="color:rgb(0, 0, 0) !important;">排行榜</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：存储用户分数，实现实时排行榜。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">任务调度</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：根据任务的优先级进行排序，方便调度执行。</font>

<font style="color:rgb(28, 31, 35);">  
</font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744197519771-1704e1e8-0296-4c2a-9b33-4b28cf7065e2.png)

高级数据类型

BitMap，<font style="color:rgba(0, 0, 0, 0.85) !important;">HyperLogLog，GEO，Stream</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 9 种常见的基本数据类型应用场景汇总</font>  


+ <font style="color:rgba(0, 0, 0, 0.85) !important;">String：缓存对象、计数器、分布式锁、分布式 session 等</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">List：阻塞队列、消息队列（但是有两个问题：1. 生产者需要自行实现全局唯一 ID；2. 不能以消费组形式消费数据）等</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Hash：缓存对象、购物车等</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Set：集合聚合计算（并集、交集、差集）的场景，如点赞、共同关注、收藏等</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Zset：最典型的就是排行榜，这个也是面试中经常问到的点</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">BitMap（2.2 版新增）：主要有 0 和 1 两种状态，可以用于签到统计、用户登录态判断等</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">HyperLogLog（2.8 版新增）：海量数据基数统计的场景，有一定的误差，可以根据场景选择使用，常用于网页 PV、UV 的统计</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">GEO（3.2 版新增）：存储地理位置信息的场景，比如说百度地图、高德地图、附近的人等</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Stream（5.0 版新增）：这个主要就是消息队列了，可以实现一个简单的消息，其相比 list 多了两个特性，分别是自动生成全局唯一消息 ID 以及支持以消费组形式消费数据（同一个消息可被分发给多个单消费者和消费者组），相比 pub/sub 它是可以被持久化。</font>

---

### **<font style="color:rgb(64, 64, 64);">Redis 数据类型总表</font>**
| **<font style="color:rgb(64, 64, 64);">数据类型</font>** | **<font style="color:rgb(64, 64, 64);">特点</font>** | **<font style="color:rgb(64, 64, 64);">使用场景</font>** | **<font style="color:rgb(64, 64, 64);">底层实现</font>** |
| --- | --- | --- | --- |
| **<font style="color:rgb(64, 64, 64);">String（字符串）</font>** | <font style="color:rgb(64, 64, 64);">二进制安全，可存文本、数字、序列化数据</font> | <font style="color:rgb(64, 64, 64);">缓存、计数器（如浏览量）、分布式锁</font> | <font style="color:rgb(64, 64, 64);">SDS（简单动态字符串）</font> |
| **<font style="color:rgb(64, 64, 64);">List（列表）</font>** | <font style="color:rgb(64, 64, 64);">有序、可重复，支持双向操作</font> | <font style="color:rgb(64, 64, 64);">消息队列、最新消息排行、历史记录</font> | <font style="color:rgb(64, 64, 64);">QuickList（3.2+，由 ziplist + linkedlist 优化）</font> |
| **<font style="color:rgb(64, 64, 64);">Hash（哈希）</font>** | <font style="color:rgb(64, 64, 64);">键值对集合，适合存储对象</font> | <font style="color:rgb(64, 64, 64);">用户信息、商品详情</font> | <font style="color:rgb(64, 64, 64);">ziplist（小数据）或 hashtable（大数据）</font> |
| **<font style="color:rgb(64, 64, 64);">Set（集合）</font>** | <font style="color:rgb(64, 64, 64);">无序、元素唯一，支持集合运算</font> | <font style="color:rgb(64, 64, 64);">标签系统、好友关系、抽奖去重</font> | <font style="color:rgb(64, 64, 64);">intset（整数集合）或 hashtable</font> |
| **<font style="color:rgb(64, 64, 64);">Zset（Sorted Set）</font>** | <font style="color:rgb(64, 64, 64);">有序、元素唯一，按分数（score）排序</font> | <font style="color:rgb(64, 64, 64);">排行榜、延迟队列、优先级任务</font> | <font style="color:rgb(64, 64, 64);">ziplist（小数据）或</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">跳跃表（skiplist） + hashtable</font>** |
| **<font style="color:rgb(64, 64, 64);">BitMap（位图）</font>** | <font style="color:rgb(64, 64, 64);">基于 String 的位操作</font> | <font style="color:rgb(64, 64, 64);">用户签到、布隆过滤器</font> | <font style="color:rgb(64, 64, 64);">String 底层（SDS）</font> |
| **<font style="color:rgb(64, 64, 64);">HyperLogLog</font>** | <font style="color:rgb(64, 64, 64);">基数统计（去重计数），误差 0.81%</font> | <font style="color:rgb(64, 64, 64);">UV 统计、大规模去重计数</font> | <font style="color:rgb(64, 64, 64);">稀疏矩阵（Sparse）或密集矩阵（Dense）</font> |
| **<font style="color:rgb(64, 64, 64);">GEO（地理空间）</font>** | <font style="color:rgb(64, 64, 64);">存储经纬度，支持位置计算</font> | <font style="color:rgb(64, 64, 64);">附近的人、地点搜索</font> | <font style="color:rgb(64, 64, 64);">Zset 实现（经纬度编码为 score）</font> |
| **<font style="color:rgb(64, 64, 64);">Stream</font>** | <font style="color:rgb(64, 64, 64);">消息队列，支持多消费者组</font> | <font style="color:rgb(64, 64, 64);">日志收集、事件溯源</font> | <font style="color:rgb(64, 64, 64);">radix tree（基数树）</font> |


## 2.Redis为什么这么快？
<font style="color:rgba(0, 0, 0, 0.85) !important;">主要有 3 个方面的原因，分别是存储方式、优秀的线程模型以及 IO 模型、高效的数据结构。</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 将数据存储在内存中，提供快速的读写速度，相比于传统的磁盘数据库，内存访问速度快得多。</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 使用单线程事件驱动模型结合 I/O 多路复用，避免了多线程上下文切换和竞争条件，提高了并发处理效率。</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 提供多种高效的数据结构（如字符串、哈希、列表、集合等），这些结构经过优化，能够快速完成各种操作</font>

## ![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744200227048-828fdacd-3b82-4bf0-a282-051a01937e39.png)  
 3.为什么Redis设计为单线程？6.0版本为何引入多线程？
### <font style="color:rgb(0, 0, 0) !important;">回答重点</font>
#### <font style="color:rgb(0, 0, 0) !important;">单线程设计原因：</font>
1. <font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 的操作是基于内存的，其大多数操作的性能瓶颈主要不是 CPU 导致的</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">使用单线程模型，代码简便的同时也减少了线程上下文切换带来的性能开销</font>
3. <font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 在单线程的情况下，使用 I/O 多路复用模型就可以提高 Redis 的 I/O 利用率了</font>

#### <font style="color:rgb(0, 0, 0) !important;">6.0 版本引入多线程的原因：</font>
1. <font style="color:rgba(0, 0, 0, 0.85) !important;">随着数据规模的增长、请求量的增多，Redis 的执行瓶颈主要在于网络 I/O。引入多线程处理可以提高网络 I/O 处理速度。</font>

### <font style="color:rgb(0, 0, 0) !important;">扩展知识</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">我们所说的 Redis 单线程，主要指的是 Redis 网络 I/O 和键值对读写这些操作是由一个线程完成的。（持久化、集群等机制其实是有后台线程执行的）  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">不过 Redis 并不是一直都单线程的，在 4.0 之后就开始引入了多线程指令，6.0 之后便正式引入了多线程的机制，不过 这里的多线程其只是针对网络请求过程使用多线程，其对于数据读写命令的处理依旧是单线程的。</font>



## <font style="color:rgb(28, 31, 35);">4.Redis中跳表是的实现原理是什么？</font>
一般跳表<font style="color:rgb(28, 31, 35);">主要是通过多层链表来实现，底层链表保存所有元素，而每一层链表都是下一层的子集。</font>

<font style="color:rgb(28, 31, 35);"> - **插入**时，首先从最高层开始查找插入位置，然后随机决定新节点的层数，最后在相应的层中插入节点并更新指针。</font>

<font style="color:rgb(28, 31, 35);"> - **删除**时，同样从最高层开始查找要删除的节点，并在各层中更新指针，以保持跳表的结构。</font>

<font style="color:rgb(28, 31, 35);"> - **查找**时，从最高层开始，逐层向下，直到找到目标元素或确定元素不存在。查找效率高，时间复杂度为 O(logn) </font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744291808180-e7b66831-a31f-4fbc-83ed-719dc34e0ecf.png)

Redis的跳表相对于普通调表多了一个回退指针，且score可以重复；

### <font style="color:rgb(0, 0, 0);">为什么 Redis 跳表实现多了个回退指针（前驱指针）</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">回退指针主要是为了提高跳表的操作效率和灵活性。</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">例如删除节点时，通过前驱指针可以在一次遍历中找到并记录所有关联的前驱节点，无需在变更指针时再次查找前驱节点。这种设计避免了重复查找过程，简化了操作逻辑，大幅提高了删除的执行效率。</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">尤其是在频繁插入和删除的场景中，回退指针减少了节点之间指针的更新复杂度，提升性能</font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744292446692-ed4ba5fe-f80a-400b-9ab3-1e8f36719e54.png)

如果还是不理解推荐查看动画讲解：[https://www.bilibili.com/video/BV1tK4y1X7de/?spm_id_from=333.337.search-card.all.click&vd_source=9d35174f7c3d230c62ba6ea39d65beb4](https://www.bilibili.com/video/BV1tK4y1X7de/?spm_id_from=333.337.search-card.all.click&vd_source=9d35174f7c3d230c62ba6ea39d65beb4)



## 5.Redis的Hash是什么？
<font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 的 hash 是一种键值对集合，可以将多个字段和值存储在同一个键中，便于管理一些关联数据。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">特点如下：</font>  


1. <font style="color:rgba(0, 0, 0, 0.85) !important;">适合存储小数据，使用哈希表实现，能够在内存中高效存储和操作。</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">支持快速的字段操作（如增、删、改、查），非常适合存储对象的属性。</font>

### <font style="color:rgb(0, 0, 0);">Hash 底层实现解析</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">Hash 是 Redis 中的一种数据基础数据结构，类似于数据结构中的哈希表，一个 Hash 可以存储 2 的 32 次方 - 1 个键值对（约 40 亿）。底层结构需要分成两个情况：</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;"></font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 6 及之前，Hash 的底层是压缩列表加上哈希表的数据结构（ziplist + hashtable）</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 7 之后，Hash 的底层是紧凑列表（Listpack）加上哈希表的数据结构（Listpack + hashtable）</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">ziplist 和 listpack 查找 key 的效率是类似的，时间复杂度都是 O（n），其主要区别就在于 listpack 解决了 ziplist 的级联更新问题。</font>



## 6.Redis中Zset实现原理是什么？
<font style="color:rgba(0, 0, 0, 0.85) !important;">Redis 中的 ZSet（有序集合，Sorted Set）是一种由跳表（Skip List）和哈希表（Hash Table）组成的数据结构。ZSet 结合了集合（Set）的特性和排序功能，能够存储具有唯一性的成员，并根据成员的分数（score）进行排序。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">ZSet 的实现由两个核心数据结构组成：</font>

1. <font style="color:rgba(0, 0, 0, 0.85) !important;">跳表（Skip List）：用于存储数据的排序和快速查找。</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">哈希表（Hash Table）：用于存储成员与其分数的映射，提供快速查找。</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">当 Zset 元素数量较少时，Redis 会使用压缩列表（Zip List）来节省内存</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">即元素个数 ≤ zset-max-ziplist-entries（默认 128）</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">元素成员名和分值的长度 ≤ zset-max-ziplist-value（默认 64 字节）</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">如果任何一个条件不满足，Zset 将使用 跳表 + 哈希表 作为底层实现。</font>



### <font style="color:rgb(0, 0, 0);">ZSet 的插入、删除和查找操作</font>
**<font style="color:rgb(0, 0, 0) !important;">插入操作（ZADD）</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：插入操作的时间复杂度为 O (log N)，因为在跳表中查找插入位置的时间复杂度是 O (log N)，哈希表的更新操作是 O (1)。</font>

1. <font style="color:rgba(0, 0, 0, 0.85) !important;">使用哈希表存储成员和分数的映射关系，分数作为哈希表中的值。</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">同时，将成员和分数插入跳表，跳表会根据分数排序。</font>
3. <font style="color:rgba(0, 0, 0, 0.85) !important;">如果是更新操作，Redis 会在哈希表中更新成员的分数，然后在跳表中更新该成员的位置。</font>

**<font style="color:rgb(0, 0, 0) !important;">删除操作（ZREM）</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：删除操作的时间复杂度为 O (log N)，其中 N 是跳表中元素的数量。</font>

1. <font style="color:rgba(0, 0, 0, 0.85) !important;">使用哈希表删除成员的映射。</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">同时在跳表中删除该成员。</font>

**<font style="color:rgb(0, 0, 0) !important;">查找操作（ZSCORE）</font>**

1. <font style="color:rgba(0, 0, 0, 0.85) !important;">在哈希表中查找成员的分数，时间复杂度为 O (1)。</font>  


**<font style="color:rgb(0, 0, 0) !important;">范围查询操作（ZRANGE、ZREVRANGE、ZRANGEBYSCORE）</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">：</font>  


1. <font style="color:rgba(0, 0, 0, 0.85) !important;">在跳表中根据分数区间查找成员，查找时间复杂度为 O (log N + M)，其中 M 是返回的成员数量。</font>
2. <font style="color:rgba(0, 0, 0, 0.85) !important;">哈希表用于快速查找成员的分数。</font>

### <font style="color:rgb(0, 0, 0);">时间复杂度</font>
<font style="color:rgba(0, 0, 0, 0.85) !important;">ZSet 操作的核心时间复杂度是 O (log N)，因为大多数操作（如插入、删除、查找等）都需要通过跳表查找元素的位置。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">对于范围查询操作，由于返回的数据量可能很大，时间复杂度为 O (log N + M)，其中 M 是返回的元素数量。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">通过成员分数查询（如 ZSCORE）的时间复杂度为 O (1)，因为 Redis 使用哈希表来存储成员与分数的映射。</font>

## <font style="color:rgba(0, 0, 0, 0.88);">7.Redis 中如何保证缓存与数据库的数据一致性？</font><font style="color:rgb(28, 31, 35);">  
</font>
实现缓存与数据库的一致性可以采用：

1. <font style="color:rgb(28, 31, 35);">先更新缓存再更新数据库</font>
2. <font style="color:rgb(28, 31, 35);">先更新数据库再更新缓存</font>
3. <font style="color:rgb(28, 31, 35);">先删除缓存，再更新数据库，再将数据库数据同步到缓存中</font>
4. <font style="color:rgb(28, 31, 35);">先更新数据库，再删除缓存，再将数据库数据同步到缓存中</font>
5. <font style="color:rgb(28, 31, 35);">【双删策略】更新数据库之前删除一次缓存，更新数据库之后再删除一次缓存。</font>
6. <font style="color:rgb(28, 31, 35);">使用 binlog 异步更新缓存，通过监听 binlog 变化异步更新缓存。</font>

<font style="color:rgb(28, 31, 35);">并不推荐使用前面 3 种方式，可以根据业务场景来选择使用后面 3 种方式。</font>

+ <font style="color:rgb(28, 31, 35);">如果要保证的是缓存与数据库的实时一致性，那就使用先更新数据库，再删除缓存的方式，最然这种方式依旧会出现短暂的数据不一致，但是它已经尽可能的保证数据一致性了。 </font>
+ <font style="color:rgb(28, 31, 35);">如果要保证缓存与数据库的最终一致性，那就可以使用 binlog + 消息队列的方式进行，因为消息队列具有失</font>
+ <font style="color:rgb(28, 31, 35);">败重试以及顺序消费的特点，能够保证数据最终是一致的。</font>

<font style="color:rgb(28, 31, 35);">4.先更新数据库，再删除缓存</font>

+ ![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744364520481-6fafdd54-f6c2-410b-9d2d-d57524664e5d.png)

<font style="color:rgb(28, 31, 35);">5.双删策略</font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744364470948-b82d1ec6-eaf6-4a6e-a325-67658dcbc2e2.png)

6.<font style="color:rgb(28, 31, 35);">使用 binlog 异步更新缓存，通过监听 binlog 变化异步更新缓存。</font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744364594873-781e5a33-c67d-4d1c-9627-71e9a0954ab6.png)

## <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">8.Redis 中的缓存击穿、缓存穿透和缓存雪崩是什么？</font>
### 回答重点
+ **缓存击穿**：指某个热点数据在缓存中失效，导致大量请求直接访问数据库。此时，由于瞬间的高并发，可能导致数据库崩溃。
+ **缓存穿透**：指查询一个不存在的数据，缓存中没有相应的记录，每次请求都会去数据库查询，造成数据库负担加重。 
+ **缓存雪崩**：指多个缓存数据在同一时间过期，导致大量请求同时访问数据库，从而造成数据库瞬间负载激增。

### 解决方案
#### 缓存击穿：
+ 使用互斥锁，确保同一时间只有一个请求可以去数据库查询并更新缓存。
+ 热点数据永不过期。

#### 缓存穿透：
+ 使用布隆过滤器，过滤掉不存在的请求，避免直接访问数据库。
+ 对查询结果进行缓存，即使是不存在的数据，也可以缓存一个标识，以减少对数据库的请求。

#### 缓存雪崩：
+ 采用随机过期时间策略，避免多个数据同时过期。
+ 使用双缓存策略，将数据同时存储在两层缓存中，减少数据库直接请求。

## 9.<font style="color:rgba(0, 0, 0, 0.88);">Redis String 类型的底层实现是什么？（SDS）</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">Redis 中的 String 类型底层实现主要基于 SDS（Simple Dynamic String 简单动态字符串）结构，并结合 int、embstr、raw 等不同的编码方式进行优化存储。</font>

1. <font style="color:rgba(0, 0, 0, 0.88);">int：适合于整数的字符串，占用内存少，</font>
2. <font style="color:rgba(0, 0, 0, 0.88);">embstr：适用于较短的字符串，key和value可以保存在一块内存里，适合读多写少场景</font>
3. <font style="color:rgba(0, 0, 0, 0.88);">raw：适用于较长的字符串，key和value分开保存，适合经常操作大字符串场景</font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744367053269-c0ab2ca7-4c75-4ea4-ad03-b39485690a69.png)

## 10.<font style="color:rgba(0, 0, 0, 0.88);">Redis 中如何实现分布式锁？</font>  

Redis中通常使用 set ex nx 命令 + lua 脚本的方式实现的，确保多个客户端不会获取同一个资源锁的同时也确保了安全解锁（锁不会被其他客户端释放）和异常状况下锁的自动释放。

<font style="color:rgba(0, 0, 0, 0.88);">扩展知识  
</font><font style="color:rgba(0, 0, 0, 0.88);">如果基于 Redis 来实现分布锁，需要利用 set ex nx 命令 + lua 脚本。</font>

1. <font style="color:rgba(0, 0, 0, 0.88);">加锁: SET lock_key uniqueValue EX expire_time NX</font>
2. <font style="color:rgba(0, 0, 0, 0.88);">解锁: 使用 lua 脚本，先通过 get 获取 key 的 value 判断锁是否是自己加的，如果是则 del。  
</font><font style="color:rgba(0, 0, 0, 0.88);">lua脚本</font>

```plain
if redis.call("GET",KEYS[1]) == ARGV[1]
then
    return redis.call("DEL",KEYS[1])
else
    return 0
end
```

<font style="color:rgba(0, 0, 0, 0.88);">复制代码</font>

## <font style="color:rgba(0, 0, 0, 0.88);">为什么锁需要设置唯一值?</font>
<font style="color:rgba(0, 0, 0, 0.88);">  
</font><font style="color:rgba(0, 0, 0, 0.88);">为了防止锁被其他客户端误释放。  
</font><font style="color:rgba(0, 0, 0, 0.88);">例如有以下场景:</font>

1. <font style="color:rgba(0, 0, 0, 0.88);">客户端 1 加锁成功，然后执行业务逻辑，但执行的时间超过了锁的过期时间</font>
2. <font style="color:rgba(0, 0, 0, 0.88);">此时锁已经过期被释放了，客户端 2 加锁成功</font>
3. <font style="color:rgba(0, 0, 0, 0.88);">客户端 2 执行业务逻辑</font>
4. <font style="color:rgba(0, 0, 0, 0.88);">客户端 1 执行完了，执行释放锁的逻辑，即删除锁。</font>客户端 2 表示很淦，执行的好好的锁没了。  
所以每个客户端加锁（客户端可能是每个线程），需要是设置一个唯一标识，比如一个 uuid，防止锁被别的客户端误释放了。

### <font style="color:rgba(0, 0, 0, 0.88);">为什么释放锁需要 lua 脚本呢?</font>
<font style="color:rgba(0, 0, 0, 0.88);">因为释放的过程需要判断该锁是否是自己加的，这是一次查询操作，如果是就删，不是就不删。如果直接使用 redis 命令，由于 redis 中这两个操作组合是不具备原子性的，会有问题。而 lua 脚本是具有原子性的。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">为什么锁需要设置过期时间?</font>
<font style="color:rgba(0, 0, 0, 0.88);">假设某个客户端加了锁之后宕机了，锁没有设置过期机制，会使得其他客户端都无法抢到锁。也就是永远锁住了。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">单点故障问题</font>
<font style="color:rgba(0, 0, 0, 0.88);">单台 Redis 实现分布式锁存在单点故障问题，如采用主从读写分离架构，如果一个客户端在主节点上锁成功，但是主节点突然宕机，由于主从延迟导致从节点还未同步到这个锁，此时可能有另一个客户端抢到新晋升的主节点，此时会导致两个客户端抢到锁，产生了数据不一致。  
</font><font style="color:rgba(0, 0, 0, 0.88);">基于这个情况，Redis 推出了 Redlock。  
</font><font style="color:rgba(0, 0, 0, 0.88);">关于 Redlock 可以移步:</font>

## 11.<font style="color:rgba(0, 0, 0, 0.88);">Redis 的 Red Lock 是什么？你了解吗？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">Red Lock，又称为红锁，是一种分布式锁的实现方案，旨在解决在分布式环境中使用 Redis 实现分布式锁时的安全性问题。  
</font><font style="color:rgba(0, 0, 0, 0.88);">一般情况下，我们在生产环境会使用主从 + 哨兵方式来部署 Redis。  
</font><font style="color:rgba(0, 0, 0, 0.88);">如果我们正在使用 redis 分布式锁，此时发生了主从切换，但从节点上不一定已经同步了主节点的锁信息。  
</font><font style="color:rgba(0, 0, 0, 0.88);">所以新的主节点上可能没有锁的信息。此时另一个业务去加锁，一看锁还没被占，于是抢到了锁开始执行业务逻辑。  
</font><font style="color:rgba(0, 0, 0, 0.88);">此时就发生了两个竞争者同时进入临界区操作临界资源的情况，可能就会发生数据不一致的问题。  
</font><font style="color:rgba(0, 0, 0, 0.88);">所以 Redis 官方推出了红锁，避免这种状况产生。它主要解决的问题就是当部分节点发生故障也不会影响锁的使用和数据问题的产生。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">扩展知识</font>
#### <font style="color:rgba(0, 0, 0, 0.88);">红锁实现原理</font>
<font style="color:rgba(0, 0, 0, 0.88);">首先要使用红锁需要集群部署 redis，官方推荐至少 5 个实例，不需要部署从库和哨兵，仅需主库。  
</font><font style="color:rgba(0, 0, 0, 0.88);">这 5 个实例（可以更多，我们按 5 个来讲述）之间没有任何关系（不同于 redis cluster），它们之间不需要任何信息交互。  
</font><font style="color:rgba(0, 0, 0, 0.88);">客户端会对这 5 个实例依次申请锁，如果最终申请成功的数量超过半数（>=3），则表明红锁申请成功，反之失败。  
</font><font style="color:rgba(0, 0, 0, 0.88);">再来看下异常情况。假设有一台实例宕机了怎么办？实际上没任何影响，因为理论上能申请成功的数量可以达到 4，超过了半数。  
</font><font style="color:rgba(0, 0, 0, 0.88);">也因为没有主从机制，不会有同步丢失锁的问题。</font>

## <font style="color:rgba(0, 0, 0, 0.88);">12.Redis 实现分布式锁时可能遇到的问题有哪些？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
1. <font style="color:rgba(0, 0, 0, 0.88);">业务未执行完，锁已到期</font>
2. <font style="color:rgba(0, 0, 0, 0.88);">单点故障问题</font>
3. <font style="color:rgba(0, 0, 0, 0.88);">主从问题不同步问题</font>
4. <font style="color:rgba(0, 0, 0, 0.88);">网络分区问题</font>
5. <font style="color:rgba(0, 0, 0, 0.88);">时钟漂移问题</font>
6. <font style="color:rgba(0, 0, 0, 0.88);">锁的可重入性问题</font>
7. <font style="color:rgba(0, 0, 0, 0.88);">误释放锁问题</font>

### <font style="color:rgba(0, 0, 0, 0.88);">扩展知识</font>
#### <font style="color:rgba(0, 0, 0, 0.88);">业务未执行完，锁已到期</font>
<font style="color:rgba(0, 0, 0, 0.88);">为了避免持有锁的客户端崩溃或因网络问题断开连接时，锁无法被正常释放，需要给锁设置过期时间。  
</font><font style="color:rgba(0, 0, 0, 0.88);">那么就有可能出现业务还在执行，锁已到期的情况。  
</font><font style="color:rgba(0, 0, 0, 0.88);">可以设置一种续约机制（Redisson 中的看门狗机制），线程 a 在执行的时候，设置一个超时时间，并且启动一个守护线程，守护线程每隔一段时间就去判断线程 a 的执行情况，如果 a 还没有执行完毕并且 a 的时间快过期了，就重新设置一下超时时间，即继续续约。  
</font><font style="color:rgba(0, 0, 0, 0.88);">锁的过期时间的设置也需要好好评估一下。  
</font><font style="color:rgba(0, 0, 0, 0.88);">如果设置太长，业务结束了它还在阻塞的话，会影响 Redis 的性能。如果设置太短，就出现刚说的问题，因此要保证是设置一个合理的时间使得在大多数情况下任务能够在锁过期之前完成。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">单点故障问题</font>
<font style="color:rgba(0, 0, 0, 0.88);">如果 Redis 单机部署，当实例宕机或不可用，整个分布式锁服务将无法正常工作，阻塞业务的正常执行。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">主从问题</font>
<font style="color:rgba(0, 0, 0, 0.88);">如果线上 Redis 是主从 + 哨兵部署的，则分布式锁可能会有问题。  
</font><font style="color:rgba(0, 0, 0, 0.88);">因为 Redis 的主从复制过程是异步实现的，如果 Redis 主节点获取到锁之后，还没同步到其他的从节点，此时 Redis 主节点发生宕机了，这个时候新的主节点上没锁的数据，因此其他客户端可以获取锁，就会导致多个应用服务同时获取锁。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">网络分区</font>
<font style="color:rgba(0, 0, 0, 0.88);">在网络不稳定的情况下，客户端与 Redis 之间的连接可能中断，如果未设置锁的过期时间，可能会导致锁无法正常释放。如果有多个锁，还可能引发锁的死锁情况。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">时钟漂移</font>
<font style="color:rgba(0, 0, 0, 0.88);">因为 Redis 分布式锁依赖于实例的时间来判断是否过期，如果时钟出现漂移，很可能导致锁直接失效。  
</font><font style="color:rgba(0, 0, 0, 0.88);">可以让所有节点的系统时钟通过 NTP 服务进行同步，减少时钟漂移的影响。</font>

## <font style="color:rgba(0, 0, 0, 0.88);">13.</font><font style="color:rgba(0, 0, 0, 0.88);">Redis 的持久化机制有哪些？</font>Redis 提供两种主要的持久化机制：
<font style="color:rgba(0, 0, 0, 0.88);">RDB (Redis Database) 快照：</font>

+ <font style="color:rgba(0, 0, 0, 0.88);">RDB 是通过生成某一时刻的数据快照来实现持久化的，可以在特定时间间隔内保存数据的快照。</font>
+ <font style="color:rgba(0, 0, 0, 0.88);">适合灾难恢复和备份，能生成紧凑的二进制文件，但可能会在崩溃时丢失最后一次快照之后的数据。</font>

<font style="color:rgba(0, 0, 0, 0.88);">AOF (Append Only File) 日志：</font>

+ <font style="color:rgba(0, 0, 0, 0.88);">AOF 通过将每个写操作追加到日志文件中实现持久化，支持将所有写操作记录下来以便恢复。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);">数据恢复更为精确，但文件体积较大，重写时可能会消耗更多资源。</font>

<font style="color:rgba(0, 0, 0, 0.88);">Redis 4.0 新增了 RDB 和 AOF 的混合持久化机制。</font>

### 混合持久化
RDB 和 AOF 都有各自的缺点。  
如果 RDB 备份的频率低，那么丢的数据多。备份的频率高，性能影响大。  
AOF 文件虽然丢数据比较少，但是恢复起来又比较耗时。  
因此 Redis 4.0 以后引入了混合持久化，通过 aof - use - rdb - preamble 配置开启混合持久化。  
当 AOF 重写的时候（注意混合持久化是在 aof 重写时触发的）。它会先生成当前时间的 RDB 快照，将其写入新的 AOF 文件头部位置。  
这段时间主线程处理的操作命令会记录在重写缓冲区中，RDB 写入完毕后将这里的增量数据追加到这个新 AOF 文件中，最后再用新 AOF 文件替换旧 AOF 文件。  
如此一来，当 Redis 通过 AOF 文件恢复数据时，会先加载 RDB，然后再重新执行指令恢复后半部分的增量数据，这样就可以大幅度提高数据恢复的速度！

## <font style="color:rgba(0, 0, 0, 0.88);">14.Redis 主从复制的实现原理是什么</font>Redis 的主从复制是指一个 Redis 实例（主节点）可以将数据复制到一个或多个从节点（从节点），从节点从主节点获取数据并保持同步。
<font style="color:rgba(0, 0, 0, 0.88);">复制流程：</font>

+ **开始同步**<font style="color:rgba(0, 0, 0, 0.88);">：从节点通过向主节点发送 PSYNC 命令发起同步。</font>
+ **全量复制**<font style="color:rgba(0, 0, 0, 0.88);">：如果是第一次连接或之前的连接失效，从节点会请求全量复制，主节点将当前数据快照（RDB文件）发送给从节点。 </font>
+ **增量复制**<font style="color:rgba(0, 0, 0, 0.88);">：全量复制完毕后，主从之间会保持一个长连接，主节点会通过这个连接将后续的写操作传递给从节点执行，来保证数据的一致。</font>
+ ![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744598015074-4450daf1-9d72-450f-b1d9-9779da942f61.png)
+ <font style="color:rgb(28, 31, 35);">整个主从集群仅主节点可以写入，其它从节点都通过复制来同步数据，这样就能保证数据的一致性。并且对读请求分散到多个节点，提高了 Redis 的吞吐量，从一定程度上也提高了 Redis 的可用性。 </font>

### 全量同步
![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744598155607-1b6a6ca7-6c0c-4347-8ae8-ba6c222ec4e6.png)

+ runid 指的是主服务器的 run ID，从节点第一次同步不知道主节点 ID，于是传递 "?"。
+ offset 为复制进度，第一次同步值为 -1。

文字版本的流程：

+ 从节点发送 psync ？ -1 ，触发同步。
+ 主节点收到从节点的 psync 命令之后，发现 runid 没值，判断是全量同步，返回 fullresync 并带上主服务器的 runid 和当前复制进度，从服务器会存储这两个值。 
+ 主节点执行 bgsave 生成 RDB 文件，在 RDB 文件生成过程中，主节点新接收到的写入数据的命令会存储到 replication buffer 中。 
+ RDB 文件生成完毕后，主节点将其发送给从节点，从节点清空旧数据，加载 RDB 的数据。 
+ 等到从节点中 RDB 文件加载完成之后，主节点将 replication buffer 缓存的数据发送给从节点，从节点执行命令，保证数据的一致性。

待同步完毕后，主从之间会保持一个长连接，主节点会通过这个连接将后续的写操作传递给从节点执行，来保证数据的一致。

### 增量同步
主从之间的网络可能不稳定，如果连接断开，主节点部分写操作未传递给从节点执行，主从数据就不一致了。

此时有一种选择是再次发起全量同步，但是全量同步数据量比较大，非常耗时。因此 Redis 在 2.8 版本引入了增量同步（psync 其实就是 2.8 引入的命令），仅需把连接断开其间的数据同步给从节点就好了。

此时需要介绍下 repl_backlog_buffer 。

repl_backlog_buffer 是一个环形缓冲区，默认大小为 1m。主节点会将写入命令存到这个缓冲区中，但是大小有限，待写入的命令超过 1m 后，会覆盖之前的数据，因为是环形写入。

增量同步也是 psync 命令，如果主节点判断从节点传递的 runid 和主节点一致，且根据 offset 判断数据还在 repl_backlog_buffer 中，则说明可以进行增量同步。

于是就去 repl_backlog_buffer 查找对应 offset 之后的命令数据，写入到 replication buffer 中，最终将其发送给 slave 节点。slave 节点收到指令之后执行对应的命令，一次增量同步的过程就完成了。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744598313811-35196d78-3b15-481e-ae0e-96f2af64f39a.png)

<font style="color:rgb(31, 35, 41);">如果根据 offset 判断数据已经被覆盖了，此时只能触发全量同步！ 因此可以调整 repl_backlog_buffer 大小，尽量避免出现全量同步。</font>

### <font style="color:rgb(31, 35, 41);">replication buffer 和 repl_backlog_buffer 的区别</font>
<font style="color:rgb(31, 35, 41);"> replication buffer 因为不同的从节点同步速度不一样，主节点会为每个从节点都创建一个 replication buffer，它用于实时传输写命令，且大小是动态的，因为对于同步速度较慢的从服务器，需要更多的内存来缓存数据。 虽说 replication buffer 没有明确的大小限制，但是可以通过 client-output-buffer-limit 间接控制，该参数可以设置不同类型客户端（普通、从服务器、发布订阅）的输出缓冲区限制。当缓冲区大小超过限制时，Redis 会断开与客户端（从节点其实就是一个客户端）的连接。</font>

<font style="color:rgb(31, 35, 41);"> client-output-buffer-limit slave 256mb 64mb 60 </font>

<font style="color:rgb(31, 35, 41);">上述配置表示，如果从服务器的输出缓冲区大小超过 256 MB 或超过 64 MB 的时间达到 60s，Redis 将断开与从服务器的连接。 </font>

<font style="color:rgb(31, 35, 41);">repl_backlog_buffer repl_backlog_buffer 在主节点上只有一个，存储最近的写命令，用于从服务器重新连接时进行部分重同步。 </font>



## <font style="color:rgba(0, 0, 0, 0.88);">15.Redis 数据过期后的删除策略是什么？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">Redis 数据过期主要有两种删除策略，分别为定期删除、惰性删除两种：</font>

+ **定期删除**<font style="color:rgba(0, 0, 0, 0.88);">：Redis 每隔一定时间（默认是 100 毫秒）会随机检查一定数量的键，如果发现过期键，则将其删除。这种方式能够在后台持续清理过期数据，防止内存膨胀。</font>
+ **惰性删除**<font style="color:rgba(0, 0, 0, 0.88);">：在每次访问键时，Redis 检查该键是否已过期，如果已过期，则将其删除。这种策略保证了在使用过程中只删除不再需要的数据，但在不访问过期键时不会立即清除。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">扩展知识</font>
#### <font style="color:rgba(0, 0, 0, 0.88);">定期删除细节</font>
<font style="color:rgba(0, 0, 0, 0.88);">定期删除策略是 Redis 内部的一个定时任务，周期性（每 100ms）地扫描一些设置了过期时间的键。  
</font><font style="color:rgba(0, 0, 0, 0.88);">要注意，Redis 并不会一次性扫描所有设置了过期时间的键，因为这样会消耗大量的 CPU 资源。它会在每次扫描时限制扫描的时间和数量，以避免对性能产生过大的影响，因此可能会有部分键过期了没有被及时删除。  
</font><font style="color:rgba(0, 0, 0, 0.88);">每次获取 20 个 key 判断是否过期，如果发现过期的 key 占比超过 25% 则继续再拉 20 个，如果小于 25% 则停止。这里还有一个时间限制，即一次删除时间不超过 25ms，即如果发现占比超过 25% 的时候，需要判断目前是否已经花了 25ms，如果已经用了这么多长也会结束。</font>

#### <font style="color:rgba(0, 0, 0, 0.88);">惰性删除优缺点</font>
+ **优点**<font style="color:rgba(0, 0, 0, 0.88);">：可以减少 CPU 的占用，因为只有查询到了相关的数据才执行删除操作，不需要主动去定时删除。</font>
+ **缺点**<font style="color:rgba(0, 0, 0, 0.88);">：如果一直没有查询一个 Key，就有可能不会被删除，这样就容易造成内存泄漏问题。</font>

### 内存回收机制
实际上，除了这两个删除，还有一个机制也会淘汰 key，即当 Redis 内存使用达到设置的 maxmemory 限制时，会触发内存回收机制。  
此时会主动删除一些过期键和其他不需要的键，以释放内存。具体的删除策略有以下几种：

+ volatile-lru：从设置了过期时间的键中使用 LRU（Least Recently Used，最近最少使用）算法删除键。
+ allkeys-lru：从所有键中使用 LRU 算法删除键。 
+ volatile-lfu：从设置了过期时间的键中使用 LFU（Least Frequently Used，最少使用频率）算法删除键。 
+ allkeys-lfu：从所有键中使用 LFU 算法删除键。 
+ volatile-random：从设置了过期时间的键中随机删除键。 
+ allkeys-random：从所有键中随机删除键。 
+ volatile-ttl：从设置了过期时间的键中根据 TTL（Time to Live，存活时间）删除键，优先删除存活时间短的键。 
+ noeviction：不删除键，只是拒绝写入新的数据。

Redis 在正常情况下对过期键的处理就是惰性删除 + 定期删除一起使用，主动删除（内存回收）其实属于异常的兜底处理了。

### Redis 键过期时间的设置
+ EXPIRE：设置键的过期时间（以秒为单位）。 
+ PEXPIRE：设置键的过期时间（以毫秒为单位）。 
+ SETEX：在设置键值的同时定义过期时间。 
+ PSETEX：类似于 SETEX，但支持毫秒级的过期时间。

## 16.<font style="color:rgba(0, 0, 0, 0.88);">如何解决 Redis 中的热点 key 问题？</font>
Redis 中的热点 Key 问题是指某些 Key 被频繁访问，导致 Redis 的压力过大，进而影响整体性能甚至导致集群节点故障。

解决热点 Key 问题的主要方法包括：

+ **热点 key 拆分**：将热点数据分散到多个 Key 中，例如通过引入随机前缀，使不同用户请求分散到多个 Key，多个 key 分布在多实例中，避免集中访问单一 Key。
+ **多级缓存**：在 Redis 前增加其他缓存层（如 CDN、本地缓存），以分担 Redis 的访问压力。 
+ **读写分离**：通过 Redis 主从复制，将读请求分发到多个从节点，从而减轻单节点压力。 
+ **限流和降级**：在热点 Key 访问过高时，应用限流策略，减少对 Redis 的请求，或者在必要时返回降级的数据或空值。

### 热点 key 的拆分
我们可以按照不同场景做不同的 “拆分“。有些场景可以全量拷贝，即将 mianshiya 这个 key 复制成 mianshiya_1、mianshiya_2、mianshiya_3 ，它们之间的数据是一致的，这样不同用户都访问到全量的数据。

有些场景直接进行 key 的拆分，mianshiya_1、mianshiya_2、mianshiya_3 各存一部分的数据，不同用户仅需访问不同数据即可，例如一些推流信息，因为一个热点往往有很多发布者，大家看一部分，后续热度稍微降低下来，可以替换数据。

不同用户可以进行 hash，将用户 id 哈希之后取余得到后缀，拼上 mianshiya_ 即可组成一个 key。

## 17.<font style="color:rgba(0, 0, 0, 0.88);">Redis 集群的实现原理是什么？</font>
### <font style="color:rgba(0, 0, 0, 0.88);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);">Redis 集群（Redis cluster）是通过多个 Redis 实例组成的，每个实例存储部分的数据（即每个实例之间的数据是不重复的）。</font>

<font style="color:rgba(0, 0, 0, 0.88);">具体是采用哈希槽（Hash Slot）机制来分配数据，将整个键空间划分为 16384 个槽（slots）。每个 Redis 实例负责一定范围的哈希槽，数据的 key 经过哈希函数计算后对 16384 取余即可定位到对应的节点。</font>

<font style="color:rgba(0, 0, 0, 0.88);">客户端在发送请求时，会通过集群的任意节点进行连接，如果该节点存储了对应的数据则直接返回，反之该节点会根据请求的键值计算哈希槽并路由到正确的节点。</font>

<font style="color:rgba(0, 0, 0, 0.88);">简单来说，集群就是通过多台机器分担单台机器上的压力。</font>

<font style="color:rgba(0, 0, 0, 0.88);">（如果不理解集群原理，可以看下扩展知识中的两个示例）</font>

### <font style="color:rgba(0, 0, 0, 0.88);">扩展知识</font>
#### <font style="color:rgba(0, 0, 0, 0.88);">Redis 集群中节点之间的信息如何同步？</font>
<font style="color:rgba(0, 0, 0, 0.88);">Redis 集群内每个节点都会保存集群的完整拓扑信息，包括每个节点的 ID、IP 地址、端口、负责的哈希槽范围等。  
</font><font style="color:rgba(0, 0, 0, 0.88);">节点之间使用 Gossip 协议进行状态交换，以保持集群的一致性和故障检测。每个节点会周期性地发送 PING 和 PONG 消息，交换集群信息，使得集群信息得以同步。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">Gossip 的优点：</font>
+ **快速收敛**<font style="color:rgba(0, 0, 0, 0.88);">：Gossip 协议能够快速传播信息，确保集群状态的迅速更新。</font>
+ **降低网络负担**<font style="color:rgba(0, 0, 0, 0.88);">：由于信息是以随机节点间的对话方式传播，避免了集中式的状态查询，从而降低了网络流量。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">Gossip 协议</font>
#### <font style="color:rgba(0, 0, 0, 0.88);">Gossip 主要特点：</font>
1. **分布式信息传播**<font style="color:rgba(0, 0, 0, 0.88);">：每个节点定期向其他节点发送其状态信息，确保所有节点对集群的状态有一致的视图。</font>
2. **低延迟和高效率**<font style="color:rgba(0, 0, 0, 0.88);">：Gossip 协议设计为轻量级的通信方式，能够快速传播信息，减少单点故障带来的风险。 </font>
3. **去中心化**<font style="color:rgba(0, 0, 0, 0.88);">：没有中心节点，所有节点平等地参与信息传播，提高了系统的鲁棒性。</font>

#### <font style="color:rgba(0, 0, 0, 0.88);">工作原理：</font>
1. **状态报告**<font style="color:rgba(0, 0, 0, 0.88);">：每个节点在特定的时间间隔内，向随机选择的其他节点发送其自身的状态信息，包括节点的主从关系、槽位分布等。 </font>
2. **信息更新**<font style="color:rgba(0, 0, 0, 0.88);">：接收到状态信息的节点会根据所接收到的数据更新自己的状态，并将更新后的状态继续传播给其他节点。 </font>
3. **节点检测**<font style="color:rgba(0, 0, 0, 0.88);">：通过周期性交换状态信息，节点可以检测到其他节点的存活状态。如果某个节点未能在预定时间内响应，则该节点会被标记为故障节点。 </font>
4. **容错处理**<font style="color:rgba(0, 0, 0, 0.88);">：在检测到节点故障后，集群中的其他节点可以采取措施（如重新分配槽位）以保持系统的高可用性。</font>

### <font style="color:rgba(0, 0, 0, 0.88);">Redis 集群分片原理图示</font>
<font style="color:rgba(0, 0, 0, 0.88);">Redis 集群会将数据分散到 16384（2 ^ 14）个哈希槽中，集群中的每个节点负责一定范围的哈希槽，在 Redis 集群中，使用 CRC16 哈希算法计算键的哈希槽，以确定该键应存储在哪个节点。  
</font><font style="color:rgba(0, 0, 0, 0.88);">集群哈希槽分片如下图所示：</font>

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744624968452-e7336f72-bfab-440c-b5d5-68287ff7116b.png)每个节点会拥有一部分的槽位，然后对应的键值会根据其本身的 key，映射到一个哈希槽中，其主要流程如下：

+ 根据键值的 key，按照 CRC 16 算法计算一个 16 bit 的值，然后将 16 bit 的值对 16384 进行取余运算，最后得到一个对应的哈希槽编号。
+ 根据每个节点分配的哈希槽区间，对应编号的数据落在对应的区间上，就能找到对应的分片实例。

### Redis 集群中存储 key 示例
假设我们有一个 Redis 集群，包含三个主节点（Node1、Node2、Node3），它们分别负责以下哈希槽：

+ Node1: 哈希槽 0-5460
+ Node2: 哈希槽 5461-10922
+ Node3: 哈希槽 10923-16383

现在要存储一个键为 user:1001 的数据。

### 计算哈希槽
1. 使用 CRC16 哈希算法计算 user:1001 的 CRC16 值。
2. 假设计算结果为 12345。
3. 然后，计算该值对应的哈希槽：
    - 哈希槽 = 12345 % 16384 = 12345。

### 确定目标节点
+ 12345 落在 Node3 的负责范围（10923-16383），因此，user:1001 会被存储在 Node3 中。

## 18.<font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">Redis 中的 Big Key 问题是什么？如何解决？</font>
### <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">回答重点</font>
<font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">Redis 中的 “big Key” 是指一个内存空间占用比较大的键（Key），它有什么危害呢？</font>

+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">内存分布不均。在集群模式下，不同 slot 分配到不同实例中，如果大 key 都映射到一个实例，则分布不均，查询效率也会受到影响。</font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">由于 Redis 单线程执行命令，操作大 Key 时耗时较长，从而导致 Redis 出现其它命令阻塞的问题。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">大 Key 对资源的占用巨大，在进行网络 I/O 传输的时候，导致获取过程中产生的网络流量较大，从而产生网络传输时间延长甚至网络传输发现阻塞的现象，例如一个 key 2MB，请求个 1000 次 2000 MB。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">客户端超时。因为操作大 Key 时耗时较长，可能导致客户端等待超时。</font>

### <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">如何解决 big key 问题？</font>
<font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">可以从以下几个方向进行入手：</font>

### <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">开发方面</font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">对要存储的数据进行压缩，压缩之后再进行存储</font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">大化小，即把大对象拆分成小对象，即将一个大 Key 拆分成若干个小 Key，降低单个 Key 的内存大小</font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">使用合适的数据结构进行存储，比如一些用 String 存储的场景，可以考虑使用 Hash、Set 等结构进行优化</font>

### <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">业务层面</font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">可以根据实际情况，调整存储策略，只存一些必要的数据。比如用户的不常用信息（地址等）不存储，仅存储用户 ID、姓名、头像等。 </font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">优化业务逻辑，从根源上避免大 Key 的产生。比如一些可以不展示的信息，直接移除等。</font>

### <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">数据分布方面</font>
+ <font style="color:rgba(0, 0, 0, 0.88);background-color:rgb(250, 250, 250);">采用 Redis 集群方式进行 Redis 的部署，然后将大 Key 拆分散落到不同的服务器上面，加快响应速度</font>



