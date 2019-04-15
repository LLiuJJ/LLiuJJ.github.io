---
title: CMU-DBMS-COURSE-05-NOTES
date: 2019-04-06 17:24:31
tags:
---

**DBMS如何管理内存，以及处理内存与磁盘间的数据交换**

- 空间控制：在磁盘哪里写入页面；目标是将经常使用的页面在磁盘上尽可能的靠近。
- 时间控制：什么时候将页面读入内存；什么时候将它们写入磁盘？目标是减少从磁盘读取数据导致的超长耗时。

**磁盘式数据库系统的架构**

![DISK-ORIENTED](CMU-DBMS-COURSE-05-NOTES/DISK-ORIENTED.png)

**缓冲池组织**

- 缓冲池是以固定大小的页以数组的形式组织成的存储区域，一个数组元素叫做帧(frame)。
- 当DBMS请求一个页面进入内存的时候，缓冲池 (Buffer Pool) 中会出现页面的精确拷贝。

**缓冲池元数据**

- 页表(Page Table)记录了当前在内存中的页面
- 同时也包括了每个页面附加的元数据：是否脏页的标志；Pin（别针）/Reference Counter（引用计数器）

![Buffer-pool-meta-data](CMU-DBMS-COURSE-05-NOTES/Buffer-pool-meta-data.png)

- 用Pin来标识页面被访问了

![Pin](CMU-DBMS-COURSE-05-NOTES/Pin.png)

**读页面的时候Page Table加锁**

![read-disk1](CMU-DBMS-COURSE-05-NOTES/read-disk1.png)

![read-disk2](CMU-DBMS-COURSE-05-NOTES/read-disk2.png)

![read-disk3](CMU-DBMS-COURSE-05-NOTES/read-disk3.png)

**锁 vs 插销 (LOCKS LATCHES)**

- 锁：保护数据库的逻辑内容不受其他事务的影响；保证事务的持久化；需要能够支持回改变化
- 插销：保护DBMS的内部数据结构的关键部分不受其他的线程影响；支持操作持久化；不需要支持回滚变化，用mutex实现

**Page table vs page directory （页表和页目录）**

- 页目录是页 id 到页在文件位置的映射，所有的更改必须记录在磁盘上，以便DBMS在重新启动时查找。
- 页表是从页 id 到缓冲池中页副本位置的映射，这是一种不需要存储在磁盘上的数据结构。

**多个缓冲池**

- 在整个系统中，DBMS不总是只有一个缓冲池：多个缓冲池实例；每个数据库的缓冲池；每个页面类型的缓冲池
- 有助于减少闩(latch)锁争用，并该静位置

**预取(PRE-PETCHING)**

- DBMS可以基于一个查询计划做预读，顺序扫描，索引扫描

![pre1](CMU-DBMS-COURSE-05-NOTES/pre1.png)

![pre2](CMU-DBMS-COURSE-05-NOTES/pre2.png)

![pre3](CMU-DBMS-COURSE-05-NOTES/pre3.png)

![pre4](CMU-DBMS-COURSE-05-NOTES/pre4.png)

![pre5](CMU-DBMS-COURSE-05-NOTES/pre5.png)

![pre6](CMU-DBMS-COURSE-05-NOTES/pre6.png)

**索引预读**

![index-pre1](CMU-DBMS-COURSE-05-NOTES/index-pre1.png)

![index-pre2](CMU-DBMS-COURSE-05-NOTES/index-pre2.png)

![index-pre3](CMU-DBMS-COURSE-05-NOTES/index-pre3.png)

![index-pre4](CMU-DBMS-COURSE-05-NOTES/index-pre4.png)

**扫描共享 (SCAN SHARING)**

- 多个查询任务能够重用从存储或操作计算中检索到的数据：这个结果缓存不同
- 允许将多个查询附加到扫描表的单个光标上：查询任务不必完全相同；中间结果也可以共享
- 如果一个查询启动了一个扫描，并且已经有一个执行了扫描，那么DBMS将附加到第二个查询的光标上：DBMS跟踪第二个查询和第一个查询的连接位置，以便在到达数据结构末尾时完成扫描
- IBMDB2 和 MSSQL完全支持
- Oracle仅支持相同查询的光标共享

![scan1](CMU-DBMS-COURSE-05-NOTES/scan1.png)

![scan2](CMU-DBMS-COURSE-05-NOTES/scan2.png)

![scan3](CMU-DBMS-COURSE-05-NOTES/scan3.png)

![scan4](CMU-DBMS-COURSE-05-NOTES/scan4.png)

![scan5](CMU-DBMS-COURSE-05-NOTES/scan5.png)

![scan6](CMU-DBMS-COURSE-05-NOTES/scan6.png)

![scan7](CMU-DBMS-COURSE-05-NOTES/scan7.png)

![scan8](CMU-DBMS-COURSE-05-NOTES/scan8.png)

![scan9](CMU-DBMS-COURSE-05-NOTES/scan9.png)

![scan10](CMU-DBMS-COURSE-05-NOTES/scan10.png)

**缓冲池旁路(Buffer pool bypass)**

- 顺序扫描运算符不会将提取的页存储在缓冲池中，以避免开销：内存是本地运行查询的；如果操作需要读取磁盘上连续的大量页序列，可以以很好的性能工作
- 在Informix中叫做"Light Scans"

**操作系统页面缓存**

- 大多数磁盘操作都是通过操作系统API进行的。除非你告诉它不要这样做，否则操作系统维护自己的文件系统缓存。
- 大多数DBMS使用直接I/O (O_direct) 绕过操作系统缓存：避免页的荣誉副本；不同的页面回收策略

**缓冲区替换策略**

- 当DBMS需要释放一个页帧(frame)来为新页面腾出空间时，它必须决定从缓冲池中退出哪个页面
- 策略目标：正确性；准确度；速度；元数据开销

**LRU（least recently used）**

- 维护每个页面上次访问的时间戳
- 当DBMS需要回收一个页面时，选择时间戳最早的页面
- 保持页面的排序顺序，以减少还出时的搜索时间

**时钟 CLOCK**

- 接近LRU但不需要每页单独的时间戳：每个页面有一个引用位；当页面被访问的时候，设置成1
- 用"时钟指针(clock hand)"在圆形缓冲区中组织页面：扫描时检查页位是否设置为1，如果是，设置成0。不是的话，换出

![clock1](CMU-DBMS-COURSE-05-NOTES/clock1.png)

![clock2](CMU-DBMS-COURSE-05-NOTES/clock2.png)

![clock3](CMU-DBMS-COURSE-05-NOTES/clock3.png)

![clock4](CMU-DBMS-COURSE-05-NOTES/clock4.png)

![clock5](CMU-DBMS-COURSE-05-NOTES/clock5.png)

![clock6](CMU-DBMS-COURSE-05-NOTES/clock6.png)

![clock7](CMU-DBMS-COURSE-05-NOTES/clock7.png)

![clock8](CMU-DBMS-COURSE-05-NOTES/clock8.png)

- 存在的问题：LRU和时钟替换策略容易受到缓冲区连续溢出的影响
- 查询执行顺序扫描，读取每一页
- 这会用一次又一次的读取的页污染缓冲池
- 这种情况下最近使用的页面实际上是最不需要的页面

**顺序溢出(Sequential flooding)**

![seq1](CMU-DBMS-COURSE-05-NOTES/seq1.png)

![seq2](CMU-DBMS-COURSE-05-NOTES/seq2.png)

![seq3](CMU-DBMS-COURSE-05-NOTES/seq3.png)

![seq4](CMU-DBMS-COURSE-05-NOTES/seq4.png)

![seq5](CMU-DBMS-COURSE-05-NOTES/seq5.png)

![seq6](CMU-DBMS-COURSE-05-NOTES/seq6.png)

**更好的策略：LRU-K**

- 考虑最后k个引用的历史作为时间戳，并计算后续访问之间的时间间隔
- 然后，DBMS使用此历史记录来估计下一次访问该页的时间

**更好的策略：本地化**

- DBMS根据 txn/查询 选择要换出的页面。这样可以最大限度地减少每个查询对缓冲池的污染
- 跟踪查询访问的页面

**更好的策略：优先级提示**

- 当查询执行的时候，DBMS知道每个页面的具体内容
- 它可以在buffer pool中提供提示，辨别一个页面是不是重要的

![index-page1](CMU-DBMS-COURSE-05-NOTES/index-page1.png)

![index-page2](CMU-DBMS-COURSE-05-NOTES/index-page2.png))

![index-page3](CMU-DBMS-COURSE-05-NOTES/index-page3.png))

**脏页**

- 快速：如果缓冲池中的一个页面不是脏的，那么DBMS可以简单地 “删除” 它
- 慢速：如果一个页面脏了，那么DBMS必须写回磁盘以确保其更改被持久化
- 我们需要在快速回收与将来不再读取的脏写页面之间进行权衡

**后台写入**

- DBMS可以周期性的遍历页表并将脏页写入磁盘
- 当安全地写入脏页时，DBMS可以收回改页或只是取消设置脏页标志
- 需要注意的是，在写入脏页的日志记录之前，我们不会写入它们

**其他的内存池**

- 除了元组和索引之外，DBMS还需要内存
- 这些其他的内存池可能并不总是由磁盘支持，这取决于具体实现
  - Sorting + Join Buffers
  - Query Caches
  - Maintenance Buffers
  - Log Buffers
  - Dictionary Caches