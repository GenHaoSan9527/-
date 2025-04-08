<h2 id="rsc6W">**为什么MySQL用B+树做索引结构？**</h2>
****

主要有三个方面：

一、高效的查询性能

B+树是一种多路平衡查找树，他的插入，删除，查询的时间复杂度都是**O(logn)**，每次插入删除都会进行自动调节平衡，让根节点到没个叶子结点的路径相同。

二、树的高度不会增长过快，从而使磁盘的I/O次数减少

不像红黑树一样，数据越多，高度越高。因为B+树只将数据存储在叶子节点中，**非叶子节点只保存主键或索引值和指针**。所以每个节点能够存储更多的索引，容易命中缓存，从而减少I/O次数。

三、查询范围强

因为B+树叶子节点采用双向链表结构连接，当从根节点定位到叶子节点后，可以顺序查寻，遍历其他数据。

猜测提问：那为什不使用B树呢？

因为B树每个节点都存储有完整数据，B+树非叶子节点只存储指针和key，能容下更多索引页，从而进一步减少I/O次数，还有B+树采用叶子采用双向链表，更适合于区间查询，而B树只能每一层遍历查找

<h2 id="Rq0Bm">**MySQL索引的最左侧前缀匹配原则是什么？**</h2>


在使用联合索引时，查询条件必须按照索引的顺序，从左往右来。

比如有个联合索引（a，b，c）然后查询条件是a = 1 and b=2 and c=3,这样不能打乱顺序，否则无法使           用。如果是a =1,b = 3，这样也是不行的。在5.6之前只能使用a = 1,查询。但是之后就有了索引下推，把           查 推到引擎层面处理之后值返回到server层。

注意范围查询当遇到（>,<）就会停止使用联合查询（停止后后续的条件会使用过滤），（>=,<=,like，      between）不会停止。停止匹配：在B+树中<font style="color:rgb(64, 64, 64);">联合索引 </font>`<font style="color:rgb(64, 64, 64);">(a, b, c, d)</font>`<font style="color:rgb(64, 64, 64);"> 在B+树中的排列方式是：</font>

+ <font style="color:rgb(64, 64, 64);">首先按a列排序</font>
+ <font style="color:rgb(64, 64, 64);">a相同则按b列排序</font>
+ <font style="color:rgb(64, 64, 64);">b相同则按c列排序</font>

最后按d列排序

如果出现（>,<），就会导致后面的查询条件无序。

8之后出现了优化，可以存在跳过扫描范围的局限性，从而使用上联合索引。

<h2 id="Lv9qo">**MySQL三层B+树能存多少数据？**</h2>


首先B+树中每个叶子节点表示一个数据页，然后这个页的大小可以设置(常规页大小：4KB、8KB、16KB、32KB、64KB）,默认是16kb要知道，每个非叶子节点存储指针和索引，指针为6b,索引一般为bigint类型所以为8b,一起就是14b,所以一个根节点就可以存储16*1024/14=1170个（约等于），然后就对应1170个中间节点，每个中间节点对应1170个叶子节点，然后假设存储的完整数据是1kb，那么一个叶子节点就可以存储16个完整数据，所以在每条数据为1kb,数据页为16kb,索引采用Bigint，的情况下存储1170*1170*16约等于2000万条数据。

<h2 id="VF8Wp">**MySQL中的回表是什么？**</h2>
简单来说就是查询时使用了二级索引（非聚族索引）作为查询条件，由于二级索引中只存储了索引值和主键，并没有完整的数据，所以拿到主键后还要去主键索引（聚族索引）查询一次，这个过程叫做回表。回表会产生随机I/O，所以会导致查询效率低。

什么操作可以减少回表操作呢？

1.尽量减少不必要的查询字段。

2.使用覆盖索引（就是索引字段包含了所有查询字段），就不需要回表。

3.还有一种情况是优化器自带索引下推，也可以避免回表。

<h2 id="TSg5L">**MySQL中使用索引一定有效吗？如何排查索引效果？**</h2>


不一定。

MySQL索引失效的核心场景：

1.破坏最左侧匹配原则

2.破坏索引覆盖原则

3.破坏前缀匹配原则

4.order by 排序不当

5.索引列上有函数或者计算

6.使用not in和not exist 不当

7.列的对比，in字段

在查询语句前带上explain 可以给出是否使用索引，使用了那个索引。type(访问类型)

| **类型** | **性能** | **说明** |
| --- | --- | --- |
| <font style="color:rgb(64, 64, 64);">system</font> | <font style="color:rgb(64, 64, 64);">★★★★★</font> | <font style="color:rgb(64, 64, 64);">最佳情况，表只有一行</font> |
| <font style="color:rgb(64, 64, 64);">const</font> | <font style="color:rgb(64, 64, 64);">★★★★☆</font> | <font style="color:rgb(64, 64, 64);">主键/唯一索引的等值查询</font> |
| <font style="color:rgb(64, 64, 64);">eq_ref</font> | <font style="color:rgb(64, 64, 64);">★★★★☆</font> | <font style="color:rgb(64, 64, 64);">连接中使用主键/唯一索引</font> |
| <font style="color:rgb(64, 64, 64);">ref</font> | <font style="color:rgb(64, 64, 64);">★★★☆☆</font> | <font style="color:rgb(64, 64, 64);">非唯一索引的等值查询</font> |
| <font style="color:rgb(64, 64, 64);">range</font> | <font style="color:rgb(64, 64, 64);">★★☆☆☆</font> | <font style="color:rgb(64, 64, 64);">索引范围扫描</font> |
| <font style="color:rgb(64, 64, 64);">index</font> | <font style="color:rgb(64, 64, 64);">★☆☆☆☆</font> | <font style="color:rgb(64, 64, 64);">全索引扫描</font> |
| <font style="color:rgb(64, 64, 64);">ALL</font> | <font style="color:rgb(64, 64, 64);">☆☆☆☆☆</font> | <font style="color:rgb(64, 64, 64);">全表扫描</font> |


key(具体使用的索引名称)，<font style="color:rgb(64, 64, 64);">当 </font>`<font style="color:rgb(64, 64, 64);">type</font>`<font style="color:rgb(64, 64, 64);"> 为 </font>`<font style="color:rgb(64, 64, 64);">ALL</font>`<font style="color:rgb(64, 64, 64);"> 或 </font>`<font style="color:rgb(64, 64, 64);">index</font>`<font style="color:rgb(64, 64, 64);"> 时，</font>`<font style="color:rgb(64, 64, 64);">key</font>`<font style="color:rgb(64, 64, 64);"> 可能为 NULL 或显示全索引扫描的索引</font>



<h2 id="mfCYq">MySQL中索引建立时需要注意那些事项？</h2>
1.索引不是越多越好，维护索引需要消耗一定资源。

2.大量重复字段不要创建索引，像性别，判断。索引在这方面的性能提升有限。

3.长字段不因该创建索引，像text，longtext，扫描时消耗内存大，可能导致整体性能下降。

4.当数据表的修改频率远大于查询频率时不因该创建索引，因为建立索引会减慢修改的效率。

5.当某些字段频繁被作为条件查询时，可以将这些字段做成联合索引，可以减少索引数量。

6.对于频繁作为order by,group by,distinct,后面的字段（一般为主键），可以作为索引，加快排序去重，分组速度。

<h2 id="LGeyZ">MySQL中的索引数量是否越多越好？为什么？</h2>
不是

1.从时间上看，每次对表的增删改，索引也要接着修改，如果数据行很多这就需要消耗很多时间，因为当删除或者增加一个数据行时，会破坏B+树的有序性，从而会自动进行分裂合并，保持有序性。

2.还有如果索引过多，也会影响优化器的选择，就像如果索引过多需要时间计算那个更优，如果索引未及时更新可能就会被pass掉。从而选择次优的索引。

3.从空间上来看，每建立一个二级索引，都会创建一个B+树，每个数据页大概占16kb,如果量和大这个占用空间可不小。



<h2 id="Q4j7c">如何使用MySQL的explain的语句进行查询分析？</h2>
![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1743678283258-98f2106b-5493-4d48-b197-1f32d9af98a8.png)

1.id:查询的执行顺序的标识符，值越大优先级越高。简单查询的id一般为1，复杂查询的话id会有多个。

2.select_type:查询的类型，如simple（简单查询），primary（主查询），subquery（子查询）等

3.table：查询数据表

4.type:访问类型const>eq_ref>ref>range>index>all

5.possible_keys:可能用到的索引

6.key:实际用到的索引

7.key_len:用到的索引

8.ref:显示索引的那一列被使用

9.rows:估计要扫描的行数，值越小越好。

10.extra:额外信息，如usering index(表示使用覆盖索引)，using where(表示使用where条件过滤)，using temporary(表示使用临时表)，using filesort（表示额外的排序步骤）。

<h2 id="WgEoe">MySQL中如何进行SQL调优？</h2>
1.平时进行SQL调优，主要是通过观察慢SQL，然后利用explain分析查询语句据的执行情况，识别瓶颈，优化查询语句。

2.合理设计索引，避免回表。

3.避免使用*查询

4.不要使用函数

5.不要使用模糊查询

6.满足最左侧匹配原则

7.不要对无索引字段进行排序操作

8.连表查询需要注意不同字段的字符集是否一致，否则会进行全表查询。

9.可以利用缓存优化，一些变化少或者访问频繁的数据设置到缓存中。

慢SQL

<font style="color:rgb(64, 64, 64);">MySQL通过</font>`<font style="color:rgb(64, 64, 64);">long_query_time</font>`<font style="color:rgb(64, 64, 64);">参数(默认10秒)来定义慢查询：</font>

+ <font style="color:rgb(64, 64, 64);">执行时间超过此阈值的SQL会被记录到慢查询日志中</font>
+ <font style="color:rgb(64, 64, 64);">实际应用中，通常会将阈值设置为更小的值(如1-2秒)</font>

```plain
-- 查看慢查询日志配置
SHOW VARIABLES LIKE '%slow_query_log%';
SHOW VARIABLES LIKE '%long_query_time%';

-- 启用慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2; -- 设置为2秒
```

1. **<font style="color:rgb(64, 64, 64);">性能分析工具</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - `<font style="color:rgb(64, 64, 64);">EXPLAIN</font>`<font style="color:rgb(64, 64, 64);">分析执行计划</font>
    - `<font style="color:rgb(64, 64, 64);">SHOW PROFILE</font>`<font style="color:rgb(64, 64, 64);">查看详细执行信息</font>
    - <font style="color:rgb(64, 64, 64);">Performance Schema监控</font>

<h2 id="qzLz4">请详细描述MySQL的B+树中的查询过程？</h2>
1.数据从根节点开始，首先你知道数据行的主键，然后将主键与根节点的索引表进行二分区间查找，从上到下，从根节点到中间结点，最后到叶子节点。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1743753537667-0d5903bd-42df-4102-a948-52c17bf198f1.png)

2.到了叶子节点并没有结束，每个叶子节点有16kb,能够存储很多数据行， 叶子节点里面有页目录，页目录由一个个槽组成，标号是从0到n,每个槽对应一个分组，用指针指向对应分组中的内存最大行主键，0槽指向最小记录组，从上到下依次。然后再次用主键在页目录中进行二分，找到对应的组，如果刚好需要查询的数据行是当前页的最大记录行那么这次查询结束。如果不是，则返回当前槽的上一位槽进行查询，从上一个分组的最大行向下遍历查找，因为每组中数据行使用单向链表链接从上到下，每组之间也是使用链表链接。

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1743753589620-52fbdb04-114c-49d1-8545-d4d9a2733c53.png)

扩展：每组的记录数有规定

第一个分组只记录一个记录

中间的额分组记录4-8个记录

最后记录1-8个记录

因此不必担心链表过长性能差

<h2 id="YCKrk">MySQL中count(*),count(1),count(字段名)有什么区别？</h2>
首先他们都是用来统计行数的**聚合函数**

| **函数形式** | **功能描述** | **是否统计NULL值** | **性能比较** | **使用场景** |
| --- | --- | --- | --- | --- |
| `<font style="color:rgb(64, 64, 64);">COUNT(*)</font>` | <font style="color:rgb(64, 64, 64);">统计表中所有行的数量，包括NULL值行</font> | <font style="color:rgb(64, 64, 64);">是</font> | <font style="color:rgb(64, 64, 64);">现代数据库优化后性能最好</font> | <font style="color:rgb(64, 64, 64);">需要统计表总行数时使用</font> |
| `<font style="color:rgb(64, 64, 64);">COUNT(1)</font>` | <font style="color:rgb(64, 64, 64);">统计表中所有行的数量，相当于为每行提供一个常量值1</font> | <font style="color:rgb(64, 64, 64);">是</font> | <font style="color:rgb(64, 64, 64);">与COUNT(*)性能几乎相同</font> | <font style="color:rgb(64, 64, 64);">功能同COUNT(*)，但语义稍不直观</font> |
| `<font style="color:rgb(64, 64, 64);">COUNT(字段名)</font>` | <font style="color:rgb(64, 64, 64);">统计指定字段非NULL值的行数</font> | <font style="color:rgb(64, 64, 64);">否</font> | <font style="color:rgb(64, 64, 64);">比COUNT(*)稍慢，需检查字段是否为NULL</font> | <font style="color:rgb(64, 64, 64);">需要统计某字段非NULL值的数量时使用</font> |


1. **<font style="color:rgb(64, 64, 64);">效率问题</font>**<font style="color:rgb(64, 64, 64);">：在现代数据库系统中，</font>`<font style="color:rgb(64, 64, 64);">COUNT(*)</font>`<font style="color:rgb(64, 64, 64);">和</font>`<font style="color:rgb(64, 64, 64);">COUNT(1)</font>`<font style="color:rgb(64, 64, 64);">的执行效率基本相同，因为优化器会对它们做相同处理。</font>
2. <font style="color:rgb(64, 64, 64);">**COUNT(字段名)**的特殊情况：</font>
    - <font style="color:rgb(64, 64, 64);">如果字段有索引，数据库可能会使用索引来加速计数</font>
    - <font style="color:rgb(64, 64, 64);">如果字段定义为NOT NULL，则结果与COUNT(*)相同</font>
3. **<font style="color:rgb(64, 64, 64);">最佳实践</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">统计总行数推荐使用</font>`<font style="color:rgb(64, 64, 64);">COUNT(*)</font>`<font style="color:rgb(64, 64, 64);">，语义最清晰</font>
    - <font style="color:rgb(64, 64, 64);">需要排除NULL值时使用</font>`<font style="color:rgb(64, 64, 64);">COUNT(字段名)</font>`
    - <font style="color:rgb(64, 64, 64);">在需要统计不同值的数量时使用</font>`<font style="color:rgb(64, 64, 64);">COUNT(DISTINCT 字段名)</font>`

<h2 id="dCIwg">MySQL中varchar和char有什么区别？</h2>
CHAR 和 VARCHAR 是 MySQL 中最常用的两种字符串类型，它们在存储方式、性能和适用场景上有显著区别。

| **特性** | **CHAR** | **VARCHAR** |
| --- | --- | --- |
| **存储方式** | 固定长度，分配指定长度的空间 | 可变长度，只占用实际需要的空间 |
| **存储大小** | 固定为定义的长度 | 实际长度 + 1或2字节长度前缀 |
| **最大长度** | 255字符 | 65,535字节(受行最大限制) |
| **空格处理** | 存储时会用空格填充到指定长度，检索时去除尾部空格 | 按原样存储，不自动填充或去除空格 |
| **内存分配** | 总是分配定义长度的内存 | 按实际内容分配内存 |
| **性能** | 读取速度快(固定长度) | 写入/更新效率更高(可变长度) |
| **适用场景** | 长度固定或几乎不变的数据(如MD5哈希、国家代码) | 长度变化较大的数据(如用户名、地址) |


<h3 id="sJMzQ">1. 存储机制差异</h3>
+ **CHAR(10)**：无论存储什么内容都占用10个字符空间
    - 存储"abc" → "abc " (补7个空格)
    - 总是占用10个字符的存储空间
+ **VARCHAR(10)**：根据实际内容占用空间
    - 存储"abc" → "abc" (只占用3个字符+1字节长度前缀)
    - 最大可存储10个字符，但只占用实际需要的空间

<h3 id="w5ISk">2. 性能考虑</h3>
+ **CHAR** 优势：
    - 固定长度，查询速度更快(特别是全表扫描)
    - 适合频繁查询但很少修改的列
+ **VARCHAR** 优势：
    - 节省存储空间(特别是大多数值远小于最大长度时)
    - 更新操作不会导致行迁移(除非长度超过原长度)

<h3 id="HVAng">3. 选择建议</h3>
+ 使用 **CHAR** 当：
    - 数据长度固定(如UUID、电话区号)
    - 列经常被排序或分组
    - 需要快速随机访问
+ 使用 **VARCHAR** 当：
    - 数据长度变化较大(如用户输入)
    - 存储空间有限或表中有很多字符串列
    - 列很少用于排序或分组

<h3 id="KxRHk">4. 注意事项</h3>
+ MySQL 5.0.3+ 版本中，VARCHAR最大长度是65,535字节(实际受行大小限制)
+ 使用UTF-8等多字节字符集时，字符数限制会减少
+ CHAR在比较时会自动去除尾部空格，VARCHAR则保留
+ 对于非常短的字符串(如Y/N标志)，CHAR可能更高效

示例：

```plain
CREATE TABLE example (
    country_code CHAR(2),      -- 固定2字符，如'US','CN'
    username VARCHAR(50),      -- 可变长度用户名
    description VARCHAR(255)   -- 可变长度描述
);
```

<h2 id="l7R2A">MySQL是如何实现事务的？</h2>
<font style="color:rgb(64, 64, 64);">一句话来说：主要是通过锁，Redo Log,Undo Log,MVCC实现事务。</font>

<h3 id="r1fNl"><font style="color:rgb(64, 64, 64);">事务：</font></h3>
+ **<font style="color:rgb(64, 64, 64);">事务 = ACID（原子性、一致性、隔离性、持久性）</font>**
+ **<font style="color:rgb(64, 64, 64);">事务控制 =</font>****<font style="color:rgb(64, 64, 64);"> </font>**`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">BEGIN</font>**`**<font style="color:rgb(64, 64, 64);"> </font>****<font style="color:rgb(64, 64, 64);">→ SQL 操作 →</font>****<font style="color:rgb(64, 64, 64);"> </font>**`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">COMMIT</font>**`**<font style="color:rgb(64, 64, 64);">/</font>**`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ROLLBACK</font>**`
+ **<font style="color:rgb(64, 64, 64);">隔离级别决定并发事务的可见性</font>**<font style="color:rgb(64, 64, 64);">（如</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">READ COMMITTED</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">可避免脏读）</font>
+ **<font style="color:rgb(64, 64, 64);">InnoDB 通过 Undo Log（回滚）、Redo Log（持久化）、MVCC（一致性读）、锁（隔离性）实现事务</font>**

<h3 id="07938244"><font style="color:rgb(64, 64, 64);">1. </font>**<font style="color:rgb(64, 64, 64);">事务日志（Transaction Logs）</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">Undo Log（回滚日志）</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">实现原子性的关键，记录事务修改前的数据版本。</font>
    - <font style="color:rgb(64, 64, 64);">事务回滚时，通过 Undo Log 恢复原始数据。</font>
    - <font style="color:rgb(64, 64, 64);">也用于实现 MVCC（多版本并发控制），提供一致性读。</font>
+ **<font style="color:rgb(64, 64, 64);">Redo Log（重做日志）</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">确保持久性，记录事务对页面的物理修改（顺序写入，高性能）。</font>
    - <font style="color:rgb(64, 64, 64);">数据库崩溃恢复时，重放 Redo Log 恢复未刷盘的提交数据。</font>
    - <font style="color:rgb(64, 64, 64);">采用</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">WAL（Write-Ahead Logging）</font>**<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">机制：数据页修改前先写日志。</font>
+ **<font style="color:rgb(64, 64, 64);">Binlog（归档日志）</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">服务层日志，用于主从复制和数据恢复（逻辑日志，记录 SQL 或行变更）。</font>
    - <font style="color:rgb(64, 64, 64);">在事务提交时写入（通过</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">sync_binlog</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">参数控制持久化策略）。</font>

---

<h3 id="efa809d3"><font style="color:rgb(64, 64, 64);">2.</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">锁机制</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">行锁</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">InnoDB 支持行级锁（共享锁 S、排他锁 X），减少锁冲突。</font>
    - <font style="color:rgb(64, 64, 64);">通过索引实现，无索引时会退化为表锁。</font>
+ **<font style="color:rgb(64, 64, 64);">间隙锁（Gap Lock）</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">解决幻读问题（在 RR 隔离级别下生效）。</font>
    - <font style="color:rgb(64, 64, 64);">锁定索引记录间的间隙，防止其他事务插入。</font>
+ **<font style="color:rgb(64, 64, 64);">意向锁（Intention Lock）</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">表级锁，快速判断表中是否有行被锁定（IS/IX）。</font>

---

<h3 id="4ec50bc5"><font style="color:rgb(64, 64, 64);">3.</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">多版本并发控制（MVCC）</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">实现原理</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">每行数据隐藏两个字段：</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_TRX_ID</font>**`<font style="color:rgb(64, 64, 64);">（事务ID）、</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_ROLL_PTR</font>**`<font style="color:rgb(64, 64, 64);">（回滚指针）。</font>
    - <font style="color:rgb(64, 64, 64);">通过 Undo Log 构建数据的历史版本链。</font>
    - <font style="color:rgb(64, 64, 64);">读操作根据隔离级别访问可见版本（通过</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ReadView</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">判断）。</font>
+ **<font style="color:rgb(64, 64, 64);">隔离级别支持</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - **<font style="color:rgb(64, 64, 64);">RC（读已提交）</font>**<font style="color:rgb(64, 64, 64);">：每次读生成新</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ReadView</font>**`<font style="color:rgb(64, 64, 64);">，能看到已提交的修改。</font>
    - **<font style="color:rgb(64, 64, 64);">RR（可重复读）</font>**<font style="color:rgb(64, 64, 64);">：事务首次读时生成</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ReadView</font>**`<font style="color:rgb(64, 64, 64);">，后续复用，避免不可重复读和幻读（部分场景需配合间隙锁）。</font>

---

<h3 id="4272a4bb"><font style="color:rgb(64, 64, 64);">4.</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">事务提交流程</font>**</h3>
1. **<font style="color:rgb(64, 64, 64);">写入 Undo Log</font>**<font style="color:rgb(64, 64, 64);">：记录修改前的数据。</font>
2. **<font style="color:rgb(64, 64, 64);">修改内存数据页</font>**<font style="color:rgb(64, 64, 64);">，并在 Redo Log Buffer 中记录变更。</font>
3. **<font style="color:rgb(64, 64, 64);">写入 Redo Log</font>**<font style="color:rgb(64, 64, 64);">（Prepare 阶段）：调用</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">fsync</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">确保日志落盘。</font>
4. **<font style="color:rgb(64, 64, 64);">写入 Binlog</font>**<font style="color:rgb(64, 64, 64);">（如果开启）：调用</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">fsync</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">持久化。</font>
5. **<font style="color:rgb(64, 64, 64);">Redo Log Commit 标记</font>**<font style="color:rgb(64, 64, 64);">（Commit 阶段）：完成提交。</font>

<h2 id="AKAq0">MySQL中的MVCC是什么？</h2>
<font style="color:rgb(64, 64, 64);">MVCC（Multi-Version Concurrency Control，多版本并发控制）是数据库管理系统（如 MySQL InnoDB、PostgreSQL）用来实现</font>**<font style="color:rgb(64, 64, 64);">高并发事务</font>**<font style="color:rgb(64, 64, 64);">的一种机制，它通过</font>**<font style="color:rgb(64, 64, 64);">数据多版本</font>**<font style="color:rgb(64, 64, 64);">和</font>**<font style="color:rgb(64, 64, 64);">快照读</font>**<font style="color:rgb(64, 64, 64);">来减少锁竞争，提高读性能，同时保证事务的隔离性。</font>

---

<h3 id="qdmfg">**<font style="color:rgb(64, 64, 64);">1. MVCC 的核心思想</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">同一行数据存储多个版本</font>**<font style="color:rgb(64, 64, 64);">，每个版本关联一个事务 ID（</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_TRX_ID</font>**`<font style="color:rgb(64, 64, 64);">）。</font>
+ **<font style="color:rgb(64, 64, 64);">读操作（SELECT）不阻塞写操作（UPDATE/DELETE）</font>**<font style="color:rgb(64, 64, 64);">，写操作也不阻塞读操作（通过读取历史版本实现）。</font>
+ **<font style="color:rgb(64, 64, 64);">不同事务看到的数据版本可能不同</font>**<font style="color:rgb(64, 64, 64);">（取决于事务的隔离级别和启动时间）。</font>

---

<h3 id="PdMQb">**<font style="color:rgb(64, 64, 64);">2. MVCC 的关键实现机制</font>**</h3>
<h3 id="5c817078">**<font style="color:rgb(64, 64, 64);">(1) 隐藏字段</font>**</h3>
<font style="color:rgb(64, 64, 64);">InnoDB 每行数据（记录）包含 3 个隐藏字段：</font>

| **<font style="color:rgb(64, 64, 64);">字段名</font>** | **<font style="color:rgb(64, 64, 64);">描述</font>** |
| --- | --- |
| `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_TRX_ID</font>**` | <font style="color:rgb(64, 64, 64);">最近修改该行数据的事务 ID</font> |
| `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_ROLL_PTR</font>**` | <font style="color:rgb(64, 64, 64);">回滚指针，指向 Undo Log 中的旧版本数据</font> |
| `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_ROW_ID</font>**` | <font style="color:rgb(64, 64, 64);">行 ID（如果表没有主键，InnoDB 会自动生成）</font> |


<h3 id="f82fe52e">**<font style="color:rgb(64, 64, 64);">(2) Undo Log（回滚日志）</font>**</h3>
+ <font style="color:rgb(64, 64, 64);">存储数据修改前的历史版本（用于回滚和 MVCC）。</font>

<font style="color:rgb(64, 64, 64);">形成</font>**<font style="color:rgb(64, 64, 64);">版本链</font>**<font style="color:rgb(64, 64, 64);">，通过 </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_ROLL_PTR</font>**`<font style="color:rgb(64, 64, 64);"> 可以找到所有历史版本：</font>

<h3 id="40e6854c">**<font style="color:rgb(64, 64, 64);">(3) ReadView（读视图）</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">决定事务能看到哪些版本的数据</font>**<font style="color:rgb(64, 64, 64);">，包含：</font>
    - `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">m_ids</font>**`<font style="color:rgb(64, 64, 64);">：当前活跃（未提交）的事务 ID 列表。</font>
    - `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">min_trx_id</font>**`<font style="color:rgb(64, 64, 64);">：</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">m_ids</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">中的最小事务 ID。</font>
    - `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">max_trx_id</font>**`<font style="color:rgb(64, 64, 64);">：下一个即将分配的事务 ID。</font>
    - `**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">creator_trx_id</font>**`<font style="color:rgb(64, 64, 64);">：创建该</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ReadView</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">的事务 ID。</font>
+ **<font style="color:rgb(64, 64, 64);">判断数据版本是否可见的规则</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">如果</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_TRX_ID < min_trx_id</font>**`<font style="color:rgb(64, 64, 64);">：说明该版本已提交，</font>**<font style="color:rgb(64, 64, 64);">可见</font>**<font style="color:rgb(64, 64, 64);">。</font>
    - <font style="color:rgb(64, 64, 64);">如果</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_TRX_ID > max_trx_id</font>**`<font style="color:rgb(64, 64, 64);">：说明该版本是未来事务修改的，</font>**<font style="color:rgb(64, 64, 64);">不可见</font>**<font style="color:rgb(64, 64, 64);">。</font>
    - <font style="color:rgb(64, 64, 64);">如果</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">min_trx_id ≤ DB_TRX_ID ≤ max_trx_id</font>**`<font style="color:rgb(64, 64, 64);">：</font>
        * <font style="color:rgb(64, 64, 64);">如果</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DB_TRX_ID</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">在</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">m_ids</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">中：说明该版本由未提交事务修改，</font>**<font style="color:rgb(64, 64, 64);">不可见</font>**<font style="color:rgb(64, 64, 64);">。</font>
        * <font style="color:rgb(64, 64, 64);">否则：说明该版本已提交，</font>**<font style="color:rgb(64, 64, 64);">可见</font>**<font style="color:rgb(64, 64, 64);">。</font>

<h2 id="pdt5e">MySQL的日志类型有哪些？bin log,redo log,和undo log的作用和区别是什么？</h2>
<font style="color:rgb(64, 64, 64);">MySQL 通过多种日志机制确保数据的</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">持久性、一致性、事务回滚</font>**<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">和</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">主从复制</font>**<font style="color:rgb(64, 64, 64);">，其中最重要的三种日志是</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">binlog（二进制日志）、redo log（重做日志）</font>**<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">和</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">undo log（回滚日志）</font>**<font style="color:rgb(64, 64, 64);">。它们的作用和区别如下：</font>

---

<h3 id="n2V6S">**<font style="color:rgb(64, 64, 64);">1. MySQL 日志类型概览</font>**</h3>
| **<font style="color:rgb(64, 64, 64);">日志类型</font>** | **<font style="color:rgb(64, 64, 64);">存储引擎支持</font>** | **<font style="color:rgb(64, 64, 64);">主要作用</font>** | **<font style="color:rgb(64, 64, 64);">是否持久化</font>** |
| --- | --- | --- | --- |
| **<font style="color:rgb(64, 64, 64);">binlog（二进制日志）</font>** | <font style="color:rgb(64, 64, 64);">所有引擎（InnoDB、MyISAM）</font> | <font style="color:rgb(64, 64, 64);">主从复制、数据恢复</font> | <font style="color:rgb(64, 64, 64);">可配置（</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">sync_binlog</font>**`<br/><font style="color:rgb(64, 64, 64);">）</font> |
| **<font style="color:rgb(64, 64, 64);">redo log（重做日志）</font>** | <font style="color:rgb(64, 64, 64);">InnoDB 特有</font> | <font style="color:rgb(64, 64, 64);">崩溃恢复，保证事务持久性</font> | <font style="color:rgb(64, 64, 64);">强制持久化（</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">innodb_flush_log_at_trx_commit=1</font>**`<br/><font style="color:rgb(64, 64, 64);">）</font> |
| **<font style="color:rgb(64, 64, 64);">undo log（回滚日志）</font>** | <font style="color:rgb(64, 64, 64);">InnoDB 特有</font> | <font style="color:rgb(64, 64, 64);">事务回滚、MVCC 多版本控制</font> | <font style="color:rgb(64, 64, 64);">随事务提交逐步清理</font> |


---

<h3 id="aqqet">**<font style="color:rgb(64, 64, 64);">2. 三种核心日志详解</font>**</h3>
<h3 id="185968aa">**<font style="color:rgb(64, 64, 64);">(1) binlog（二进制日志）</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">作用</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - **<font style="color:rgb(64, 64, 64);">主从复制</font>**<font style="color:rgb(64, 64, 64);">：从库通过 binlog 同步主库的数据变更。</font>
    - **<font style="color:rgb(64, 64, 64);">数据恢复</font>**<font style="color:rgb(64, 64, 64);">：通过</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">mysqlbinlog</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">工具恢复误删数据。</font>
    - **<font style="color:rgb(64, 64, 64);">审计</font>**<font style="color:rgb(64, 64, 64);">：记录所有修改数据的 SQL 语句。</font>
+ **<font style="color:rgb(64, 64, 64);">特点</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - **<font style="color:rgb(64, 64, 64);">逻辑日志</font>**<font style="color:rgb(64, 64, 64);">：记录 SQL 语句或行变更（</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ROW</font>**`<font style="color:rgb(64, 64, 64);">/</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">STATEMENT</font>**`<font style="color:rgb(64, 64, 64);">/</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">MIXED</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">格式）。</font>
    - **<font style="color:rgb(64, 64, 64);">追加写入</font>**<font style="color:rgb(64, 64, 64);">：文件达到一定大小后切换新文件（不覆盖旧日志）。</font>
    - **<font style="color:rgb(64, 64, 64);">可关闭</font>**<font style="color:rgb(64, 64, 64);">：但通常建议开启（关键场景必须启用）。</font>

**<font style="color:rgb(64, 64, 64);">关键配置</font>**<font style="color:rgb(64, 64, 64);">：</font>

+ <font style="color:rgb(255, 255, 255);background-color:rgb(80, 80, 90);">ini</font><font style="background-color:rgb(80, 80, 90);">复制</font>

```plain
log_bin = ON                     # 开启 binlog
sync_binlog = 1                  # 每次事务提交都刷盘（保证不丢失）
binlog_format = ROW              # 推荐 ROW 格式（记录行变更）
expire_logs_days = 7             # 日志保留天数
```

---

<h3 id="4bfbddbb">**<font style="color:rgb(64, 64, 64);">(2) redo log（重做日志）</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">作用</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - **<font style="color:rgb(64, 64, 64);">崩溃恢复</font>**<font style="color:rgb(64, 64, 64);">：确保事务提交后数据不丢失（即使 MySQL 崩溃）。</font>
    - **<font style="color:rgb(64, 64, 64);">Write-Ahead Logging (WAL)</font>**<font style="color:rgb(64, 64, 64);">：修改数据前先写日志，再写磁盘。</font>
+ **<font style="color:rgb(64, 64, 64);">特点</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - **<font style="color:rgb(64, 64, 64);">物理日志</font>**<font style="color:rgb(64, 64, 64);">：记录数据页的物理修改（如“在 page 5 的 offset 10 写入 ‘abc’”）。</font>
    - **<font style="color:rgb(64, 64, 64);">循环写入</font>**<font style="color:rgb(64, 64, 64);">：固定大小文件组（如</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ib_logfile0</font>**`<font style="color:rgb(64, 64, 64);">、</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ib_logfile1</font>**`<font style="color:rgb(64, 64, 64);">），写满后覆盖。</font>
    - **<font style="color:rgb(64, 64, 64);">高性能</font>**<font style="color:rgb(64, 64, 64);">：顺序 I/O（比随机写数据文件快）。</font>
+ **<font style="color:rgb(64, 64, 64);">工作流程</font>**<font style="color:rgb(64, 64, 64);">：</font>
    1. <font style="color:rgb(64, 64, 64);">事务修改数据时，先写入</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">redo log buffer</font>**<font style="color:rgb(64, 64, 64);">（内存）。</font>
    2. <font style="color:rgb(64, 64, 64);">事务提交时，调用</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">fsync</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">刷盘（持久化到</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ib_logfile</font>**`<font style="color:rgb(64, 64, 64);">）。</font>
    3. <font style="color:rgb(64, 64, 64);">后台线程定期将脏页（修改的数据页）刷到磁盘。</font>

**<font style="color:rgb(64, 64, 64);">关键配置</font>**<font style="color:rgb(64, 64, 64);">：</font>

+ <font style="color:rgb(255, 255, 255);background-color:rgb(80, 80, 90);">ini</font><font style="background-color:rgb(80, 80, 90);">复制</font>

```plain
innodb_flush_log_at_trx_commit = 1  # 提交时刷盘（最安全）
innodb_log_file_size = 1G           # 单个 redo log 文件大小
innodb_log_files_in_group = 2       # redo log 文件数量
```

---

<h3 id="fa93d795">**<font style="color:rgb(64, 64, 64);">(3) undo log（回滚日志）</font>**</h3>
+ **<font style="color:rgb(64, 64, 64);">作用</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - **<font style="color:rgb(64, 64, 64);">事务回滚</font>**<font style="color:rgb(64, 64, 64);">：记录数据修改前的值，用于</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ROLLBACK</font>**`<font style="color:rgb(64, 64, 64);">。</font>
    - **<font style="color:rgb(64, 64, 64);">MVCC 多版本控制</font>**<font style="color:rgb(64, 64, 64);">：提供一致性读（如 RR 隔离级别下读取历史版本）。</font>
+ **<font style="color:rgb(64, 64, 64);">特点</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - **<font style="color:rgb(64, 64, 64);">逻辑日志</font>**<font style="color:rgb(64, 64, 64);">：记录反向 SQL（如</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">INSERT</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">对应</font><font style="color:rgb(64, 64, 64);"> </font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">DELETE</font>**`<font style="color:rgb(64, 64, 64);">，</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">UPDATE</font>**`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">对应旧值）。</font>
    - **<font style="color:rgb(64, 64, 64);">存储在系统表空间（</font>**`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">ibdata1</font>**`**<font style="color:rgb(64, 64, 64);">）或独立 undo 表空间</font>**<font style="color:rgb(64, 64, 64);">。</font>
    - **<font style="color:rgb(64, 64, 64);">随事务提交逐步清理</font>**<font style="color:rgb(64, 64, 64);">（长事务会导致 undo log 堆积）。</font>
+ **<font style="color:rgb(64, 64, 64);">工作流程</font>**<font style="color:rgb(64, 64, 64);">：</font>
    - <font style="color:rgb(64, 64, 64);">事务修改数据前，先记录 undo log。</font>
    - <font style="color:rgb(64, 64, 64);">如果事务回滚，通过 undo log 恢复数据。</font>
    - <font style="color:rgb(64, 64, 64);">如果事务提交，undo log 不会立即删除（可能被 MVCC 读取）。</font>

---

<h3 id="XEAjw">**<font style="color:rgb(64, 64, 64);">3. 三者的核心区别</font>**</h3>
| **<font style="color:rgb(64, 64, 64);">对比项</font>** | **<font style="color:rgb(64, 64, 64);">binlog</font>** | **<font style="color:rgb(64, 64, 64);">redo log</font>** | **<font style="color:rgb(64, 64, 64);">undo log</font>** |
| --- | --- | --- | --- |
| **<font style="color:rgb(64, 64, 64);">所属层级</font>** | <font style="color:rgb(64, 64, 64);">MySQL Server 层</font> | <font style="color:rgb(64, 64, 64);">InnoDB 存储引擎层</font> | <font style="color:rgb(64, 64, 64);">InnoDB 存储引擎层</font> |
| **<font style="color:rgb(64, 64, 64);">日志类型</font>** | <font style="color:rgb(64, 64, 64);">逻辑日志（SQL 或行变更）</font> | <font style="color:rgb(64, 64, 64);">物理日志（数据页修改）</font> | <font style="color:rgb(64, 64, 64);">逻辑日志（反向操作）</font> |
| **<font style="color:rgb(64, 64, 64);">作用</font>** | <font style="color:rgb(64, 64, 64);">主从复制、数据恢复</font> | <font style="color:rgb(64, 64, 64);">崩溃恢复，保证持久性</font> | <font style="color:rgb(64, 64, 64);">事务回滚、MVCC</font> |
| **<font style="color:rgb(64, 64, 64);">写入时机</font>** | <font style="color:rgb(64, 64, 64);">事务提交后</font> | <font style="color:rgb(64, 64, 64);">事务执行中（Prepare 阶段）</font> | <font style="color:rgb(64, 64, 64);">事务修改数据前</font> |
| **<font style="color:rgb(64, 64, 64);">持久化</font>** | <font style="color:rgb(64, 64, 64);">可配置（</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">sync_binlog</font>**`<br/><font style="color:rgb(64, 64, 64);">）</font> | <font style="color:rgb(64, 64, 64);">强制（</font>`**<font style="color:rgb(64, 64, 64);background-color:rgb(236, 236, 236);">innodb_flush_log_at_trx_commit=1</font>**`<br/><font style="color:rgb(64, 64, 64);">）</font> | <font style="color:rgb(64, 64, 64);">随事务提交清理</font> |
| **<font style="color:rgb(64, 64, 64);">日志清理</font>** | <font style="color:rgb(64, 64, 64);">按时间/大小过期删除</font> | <font style="color:rgb(64, 64, 64);">循环覆盖</font> | <font style="color:rgb(64, 64, 64);">事务提交后逐步清理</font> |




<h2 id="x2x2Q">数据库的脏读、不可重复读和幻读分别是什么？</h2>
1.脏读（Dirty Read）

一个事务读取到另一个事务未提交的的数据。

2.不可重复读（Non-repeatable Read）

在同一事务中，读取同一数据两次，两次值不同。

3.幻读（Phantom Read）

在同一事务中执行相同的查询操作，返回的结果集的数量不同。

发生的隔离级别：

读取未提交脏读

读取已提交可以防止脏读，但可能出现不可重复读

可重复读可以防止脏读和不可重复读，但可能出现幻读

串行化防止所有三种问题，但性能开销大。

<h2 id="TR36d">MySQL中的事务隔离级别有哪些？</h2>
四种：

1.读未提交

一个事务可以看到另一个事务尚未提交的数据修改。

2.读已提交

一个事务只能看到已经提交的其他事务所做的修改。

3.读可重复

一个事务中的多个查询返回结果是一致的。

4.串行化   

事务的操作结果相当于一个按顺序执行的单线程操作。可以避免所有并发问题，但是大大降低并发性能。

<h2 id="swy1w">MySQL默认的事务隔离级别是什么？为什么选择这个级别？</h2>
RR,可重复级别，原因是为了兼容早期binlog的statement格式问题（记录的是sql语句，事务谁先提交先执行谁），如果使用读已提交和未提交，会导致主从库不同步。RR因为这个隔离级别下有间隙锁，临键锁能够解决这个问题。

<h2 id="g3YW4">MySQL中有那些锁？</h2>
![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744030370191-477fea74-b9db-40b1-877a-39aa2e46afaf.png)

<h2 id="TUxdq">MySQL的事务二阶段提交是什么？</h2>
<font style="color:rgb(64, 64, 64);">二阶段提交是MySQL中，为了确保redolog（重做日志）和binlog(二进制日志)之间的一致性，使用的一种机制。MySQL通过二阶段提交来保证崩溃时，不会出现数据丢失或数据不一致性的情况。</font>

<font style="color:rgb(64, 64, 64);">二阶段提交的两个阶段：</font>

<font style="color:rgb(64, 64, 64);">1.准备阶段：事务提交时，将其标为准备阶段</font>

<font style="color:rgb(64, 64, 64);">2.提交阶段：写入binlog成功后变为已提交</font>

<h2 id="RMB0z">MySQL中如果发生死锁因该如何解决？</h2>
自动检测与回滚

MySQL自带死锁检测机制，当检测到死锁时会自动回滚其中一个事务，一般为持有资源最少的事务。

也有锁等待超时的参数，当获取锁的等待时间超过阈值，就会释放锁进行回滚。

手动kill发生死锁的语句

可以通过命令，手动找出被阻塞的食物及线程ID，手动kill，释放资源。

<h2 id="urrPP">MySQL中如何和解决深度分页的问题？</h2>
深度分页：就是当数据量和大的时候，按照分页访问后面的数据，列如limit 99999990 10,这样是的数据库扫描前面的99999990条数据，才能得到最终的10条数据，大批量的扫描数据会增加数据库的负载，影响性能。

解决办法：

1.子查询，利用子查询先去找到需要查询的ID（主键），一般有索引会很快，在直接通过主键查询。

2.游标分页：就是每次查询把每页最大的ID保留下来，下次直接从这里开始查询。

3.elasticsearch

<h2 id="Zhheu">什么是MySQL的主从同步机制？他是如何实现的？</h2>
MySQL中的主从同步机制是一种数据复制技术，用于将主数据库上的数据同步到一个或者多个从库当中。主要通过二进制日志（binlong）实现数据的复制。主库在执行写操作时，会将这些操作记录到日志中，然后推送给从库，从库重放对应日志文件，即可完成复制。

实现方法：

1.异步复制：主库不需要等待从库的响应（性能高，数据一致性低）

2.同步复制：主库同步等待所有从库确认收到数据（性能差，数据一致性高）

3.半同步复制：主库等待至少一个从库的确认收到数据（性能折中，数据一致性较高）（MySQL5.7有的）

异步同步：

![](https://cdn.nlark.com/yuque/0/2025/png/32795398/1744107490836-f553641a-e813-46b5-829e-960be1b94168.png)

扩展：

<font style="color:rgba(0, 0, 0, 0.85) !important;">异步复制  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">MySQL 默认是异步复制，具体流程如下：  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">主库：</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">接收到提交事务请求</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">更新数据</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">将数据写到 binlog 中</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">给客户端响应</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">主库推送 binlog 变更事件到从库中</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">从库接收到事件去主库拉取数据</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">从库：</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">由 I/O 线程将同步过来的 binlog 写入到 relay log 中。</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">由 SQL 线程从 relay log 重放事件，更新数据</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">给主库返回响应。</font>

<h3 id="9c335f96"><font style="color:rgb(0, 0, 0);">同步复制</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">主库需要将 binlog 复制到所有从库，等所有从库响应了之后才会给客户端响应，这样的话性能很差，一般不会选择同步复制。</font>

<h3 id="b00b0eb9"><font style="color:rgb(0, 0, 0);">半同步复制</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">MySQL 5.7 之后搞了个半同步复制，有个参数可以选择 “成功同步几个从库就返回响应。”  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">比如一共有 3 个从库，我参数配置 1，那么只要有一个从库响应说复制成功了，主库就直接返回响应给客户端，不会等待其他两个从库。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">这样的话性能就比较好，并且数据可靠性也增强了，只有当那个从库和主库同时都挂了，才会缺失数据。</font>

<h3 id="104672a0"><font style="color:rgb(0, 0, 0);">并行复制</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">以前从库是通过一个 SQL 线程按照顺序逐条执行主库的 binlog 日志指令（即从 relay log 重放事件）。对于主库的高并发写入操作，这种串行执行的方式会导致从库的复制速度跟不上主库，从而产生主从延迟问题。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">所以 MySQL 引入了并行复制，说白了就是通过多个 SQL 线程来并发执行重放事件。MySQL 提供了以下几种并行复制模式。</font>

<h3 id="050ccf54"><font style="color:rgb(0, 0, 0);">MySQL 5.6 基于库级别的并行复制</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">假设主库有多个数据库（如 db1 和 db2 ），在从库上，db1 的事务和 db2 的事务可以同时执行。即将不同数据库上的事务分配到不同的 SQL 线程中执行。如果主库大部分事务集中在一个数据库上，这个就没啥用了。</font>

<h3 id="9a4d8012"><font style="color:rgb(0, 0, 0);">MySQL 5.7 基于组提交（Group Commit）事务的并行复制</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">进一步细化了并行的颗粒度，从库级别细化到组提交级别。即 MySQL 会将组提交的事务视为彼此独立的事务，可以在从库并行重放。</font>

<font style="color:rgba(0, 0, 0, 0.85) !important;">如果主库开启了组提交，且事务之间没有冲突。那么这些事务都可以由多线程并行执行。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">binlog 中如果两个事务的 last_committed 相同，说明这两个事务是在同一个 Group 内提交的。但是粒度还是不够，如果大部分事务都集中在一张表上？  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">那么只有相同 last_committed 可以并发，即使有些数据的更改和当前事务是不冲突的，也无法并发。</font>

<h3 id="43fa02a5"><font style="color:rgb(0, 0, 0) !important;">MySQL 5.7 基于逻辑时钟（LOGICAL_CLOCK）的并行复制</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">LOGICAL_CLOCK 是基于 Group Commit 的并行复制，引入了时间标记的概念。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">基于所有在主库同时处于 prepare 阶段且未提交的事务不会存在锁冲突的情况（就是这些事务在从库执行时都可以并行执行），将这些事务都打上一个时间标记（实际实现用的是上一个提交事务的 sequence_number，即上面提到的 binlog 中的 last_committed ）。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">这样从库在识别到这些事务时，可以并行，进一步的提高并发度。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">所以提升主库的组提交事务数可以让从库复制的并行度更高，所以 MySQL 5.7 引入了两个参数：</font>

<font style="color:rgb(0, 0, 0);">  
</font>

+ <font style="color:rgba(0, 0, 0, 0.85) !important;">binlog_group_commit_sync_delay：组提交等待延迟提交的时间。即每次 binlog 组提交时等待一段时间再 fsync。让进入 group 的事务更多，提高并行度。</font>
+ <font style="color:rgba(0, 0, 0, 0.85) !important;">binlog_group_commit_sync_no_delay_count：组提交时允许的最大事务数量，达到该数量时立即触发组提交，而无需等待延迟时间（binlog_group_commit_sync_delay）。即达到期望的并行度后立即提交，缩小等待时长。</font>

<h3 id="d9520b1f"><font style="color:rgb(0, 0, 0) !important;">MySQL 8.0 基于 WriteSet 的并行复制</font></h3>
<font style="color:rgba(0, 0, 0, 0.85) !important;">在 MySQL 5.7 中，为了提升从库的事务回放速度，需要在主库提高事务的并行度。主库上的事务越多线程并行提交，备库就能在更大程度上实现并行回放。然而，这种方式依赖于主库的并行提交情况，当主库事务是串行提交时，备库的回放效率会显著下降。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">所以 MySQL 8.0 引入了基于 WriteSet 的并行复制，即使主库上的事务是串行提交的，只要事务之间没有冲突，备库也可以并行回放这些事务，提升复制效率。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">WriteSet 是事务更新行的集合，通过哈希算法对主键或唯一索引生成标识，记录在二进制日志中。即通过 WriteSet 判定事务之间的冲突，如果两个事务的WriteSet没有冲突，则它们可以并行回放。</font>

<h2 id="qq45a">MySQL如何处理主从同步延迟？</h2>
<font style="color:rgba(0, 0, 0, 0.85) !important;">首先需要明确一个点延迟是必然存在的，无论怎么优化都无法避免延迟的存在，只能减少延迟的时间。  
</font><font style="color:rgba(0, 0, 0, 0.85) !important;">常见解决方式有以下几种：</font>  


+ **<font style="color:rgb(0, 0, 0) !important;">二次查询</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">。如果从库查不到数据，则再去主库查一遍，由 API 封装这个逻辑即可，算是一个兜底策略，比较简单。不过等于读的压力又转移到主库身上了，如果有不法分子故意查询必定查不到的查询，这就对主库产生冲击了。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">强制将写之后立马读的操作转移到主库上</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">。这种属于代码写死了，比如一些写入之后立马查询的操作，就绑定在一起，写死都走主库。不推荐，比较死板。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">关键业务读写都走主库</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">，非关键还是读写分离。比如上面我举例的用户注册这种，可以读写主库，这样就不会有登陆报该用户不存在的问题，这种访问量频次应该也不会很多，所以看业务适当调整此类接口。</font>
+ **<font style="color:rgb(0, 0, 0) !important;">使用缓存</font>**<font style="color:rgba(0, 0, 0, 0.85) !important;">，主库写入后同步到缓存中，这样查询时可以先查询缓存，避免了延迟的问题，不过又引入了缓存数据一致性的问题。</font>  


<font style="color:rgba(0, 0, 0, 0.85) !important;">除此之外，也可以提一提配置问题，例如主库的配置高，从库的配置太低了，可以提升从库的配置等。如果面试官对 MySQL 比较熟，可能会追问一些偏 DBA 侧的问题，例如并行复制等。</font>

<h3 id="143c7cf2"><font style="color:rgb(0, 0, 0);">MySQL 主从延迟的常见原因及优化方案</font></h3>
| **<font style="color:rgb(0, 0, 0) !important;">原因</font>** | **<font style="color:rgb(0, 0, 0) !important;">优化方案</font>** |
| :--- | :--- |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">从库单线程复制</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">启用多线程复制：MySQL 并行复制。</font> |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">网络延迟</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">优化网络连接，缩短主从之间的物理距离。</font> |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">从库性能不足</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">增加硬件资源。</font> |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">长事务</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">优化应用程序，减少主库写入压力，避免长事务。MySQL 中长事务可能会导致哪些问题？</font> |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">从库太多</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">从库过多主库同步压力大，导致延迟，合理的评估主从数量</font> |
| <font style="color:rgba(0, 0, 0, 0.85) !important;">从库负载太高</font> | <font style="color:rgba(0, 0, 0, 0.85) !important;">增加从库实例，分散查询压力，避免慢查询。</font> |


<font style="color:rgba(0, 0, 0, 0.85) !important;">再强调一下主从延迟是必然存在的，无论怎么优化都无法避免延迟的存在，只能减少延迟的时间。例如即使启用多线程复制就一定可以避免延迟吗？所以业务上的解决方案还是以回答重点内的几个解决方式为准。</font>

