---
title: CMU-DBMS-COURSE-06-NOTES
date: 2019-04-06 17:24:36
tags:
---

**课程进度**

- 前面我们讲了磁盘管理和缓冲池管理，现在我们将讨论如何支持DBMS的执行引擎从页面读取/写入数据。
- 这涉及到两种重要的数据结构：哈希表（Hash Tables），树(Tree)

![status](CMU-DBMS-COURSE-06-NOTES/status.png)

- 数据库中用到这两种基础数据结构的组件：内部元数据；核心数据存储；临时数据存储；表索引

**设计上的思考**

- 数据组织：我们如何在内存/页面中布局数据结构，以及存储哪些信息以支持高效访问。
- 并发控制：如何支持多线程同时访问数据结构而不出问题。

**哈希表**

- 哈希表实现了将键映射到值的关联数组抽象数据类型。
- 它使用一个散列函数来计算数组中的偏移量，从中找到所需的值。

**静态Hash表**

- 为需要记录的每个元素分配一个槽的巨型数组。
- 去找一个元素的时候，对key用数组元素的数量取模去找到它在数组中的偏移量。

![static-hash1](CMU-DBMS-COURSE-06-NOTES/static-hash1.png)

![static-hash2](CMU-DBMS-COURSE-06-NOTES/static-hash2.png)

- 假设你事先知道元素的数量，每个key都是唯一的，有一个完美的hash函数，确保如果 key1 != key2，则 hash(key1) != hash(key2)。

**哈希函数**

- 我们不想在连接算法中使用加密哈希函数。
- 我们希望实现的哈希算法，速度快，碰撞率足够低。
- 一些有名的哈希函数
  - MurmurHash（2008）：被设计成一个快速、通用的哈希函数。
  - Google CityHash（2011）：基于MurmurHash2的设计思想；被设计成对小key(< 64 bytes)哈希速度很快。
  - Google FarmHash（2014）：Cityhash的新版本，有更低的碰撞率。
  - CLHash（2016）：基于无进位乘法的快速散列函数。
- 这些哈希函数性能对比测试

![hash-benchmark](CMU-DBMS-COURSE-06-NOTES/hash-benchmark.png)

**静态哈希函数的方案**

- 方法一：线性探针散列(Linear Probe Hashing)
- 方法二：Robin Hood hashing
- 方法三：Cuckoo Hashing

**Linear Probe Hashing**

- 一张巨大的充满了slot的表
- 通过线性的去搜索表中的下一个空闲slot来解决冲突
  - 要确定元素是否存在，请散列到索引重的某个位置并扫描它
  - 必须将key存储在索引中才能知道何时停止扫描

- 算法具体流程

![probe-hash](CMU-DBMS-COURSE-06-NOTES/probe-hash.png)

- 示例

![liner-probe1](CMU-DBMS-COURSE-06-NOTES/liner-probe1.png)

![liner-probe2](CMU-DBMS-COURSE-06-NOTES/liner-probe2.png)

![liner-probe3](CMU-DBMS-COURSE-06-NOTES/liner-probe3.png)

![liner-probe4](CMU-DBMS-COURSE-06-NOTES/liner-probe4.png)

![liner-probe5](CMU-DBMS-COURSE-06-NOTES/liner-probe5.png)

![liner-probe6](CMU-DBMS-COURSE-06-NOTES/liner-probe6.png)

**Robin Hood Hashing**

- Linear probe hashing的变种，从“富”键中窃取插槽，并将其提供给“差”键。
- 每个键跟踪它们在表中最佳位置的位置的数量
- 插入时，如果第一个key比第二个key离其最佳位置更远，则key会占用另一个key的槽
- 示例

![robin-hash1](CMU-DBMS-COURSE-06-NOTES/robin-hash1.png)

![robin-hash2](CMU-DBMS-COURSE-06-NOTES/robin-hash2.png)

![robin-hash3](CMU-DBMS-COURSE-06-NOTES/robin-hash3.png)

![robin-hash4](CMU-DBMS-COURSE-06-NOTES/robin-hash4.png)

![robin-hash5](CMU-DBMS-COURSE-06-NOTES/robin-hash5.png)

![robin-hash6](CMU-DBMS-COURSE-06-NOTES/robin-hash6.png)

![robin-hash7](CMU-DBMS-COURSE-06-NOTES/robin-hash7.png)

![robin-hash8](CMU-DBMS-COURSE-06-NOTES/robin-hash8.png)

![robin-hash9](CMU-DBMS-COURSE-06-NOTES/robin-hash9.png)

**CUCKOO HASHING**

- 使用具有不同哈希函数的多个哈希表
- 在插入时，检查每一张表并挑选任何一个有空闲slot的
- 如果没有表有空闲的slot，则其中一个表移出元素，然后重新散列以找到一个新的位置
- 查找和删除都是O(1)，因为每个hash表只检查一个位置

![cuckoo-hash1](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash1.png)

![cuckoo-hash2](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash2.png)

![cuckoo-hash3](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash3.png)

![cuckoo-hash4](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash4.png)

![cuckoo-hash5](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash5.png)

![cuckoo-hash6](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash6.png)

![cuckoo-hash7](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash7.png)

![cuckoo-hash8](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash8.png)

![cuckoo-hash9](CMU-DBMS-COURSE-06-NOTES/cuckoo-hash9.png)

- 确保移动键的时候不会陷入无限循环
- 如果我们找到一个循环，那么我们可以用新的哈希函数来重建真个哈希表
- 对于两个散列函数，我们（可能）不需要重建表，直到它满到50%
- 对于三个哈希函数，我们（可能）不需要重新构建表，直到它达到大约90%的满值

**总结**

- 前面的哈希表要求知道要提前存储元素的数量
- 如果需要增大/缩小，则需要重新生成整个表
- 动态哈希表可以能够按需增长/收缩

**链式散列**

- 维护哈希表中每个槽的桶的链接列表
- 通过将具有相同哈希键的所有元素放入同一个存储桶来解决冲突
- 要确定元素是否存在，请散列到其bucket并扫描它

![chained-hash1](CMU-DBMS-COURSE-06-NOTES/chained-hash1.png)

![chained-hash2](CMU-DBMS-COURSE-06-NOTES/chained-hash2.png)

- 哈希表可以无限增长，因为你只需不断向链接列表中添加新的存储桶
- 你只需要在bucket上使用一个闩锁来存储一个新条目或扩展链接链表

**可扩展散列**

- 我们将桶拆分，而不是让链表永远增长。
- 这要求拆分时重新调整条目，但是更改已经本地化

![extendible-hash1](CMU-DBMS-COURSE-06-NOTES/extendible-hash1.png)

![extendible-hash2](CMU-DBMS-COURSE-06-NOTES/extendible-hash2.png)

![extendible-hash3](CMU-DBMS-COURSE-06-NOTES/extendible-hash3.png)

![extendible-hash4](CMU-DBMS-COURSE-06-NOTES/extendible-hash4.png)

![extendible-hash5](CMU-DBMS-COURSE-06-NOTES/extendible-hash5.png)

![extendible-hash6](CMU-DBMS-COURSE-06-NOTES/extendible-hash6.png)

![extendible-hash7](CMU-DBMS-COURSE-06-NOTES/extendible-hash7.png)

![extendible-hash8](CMU-DBMS-COURSE-06-NOTES/extendible-hash8.png)

![extendible-hash9](CMU-DBMS-COURSE-06-NOTES/extendible-hash9.png)

**Linear Hashing**

- 保持跟踪下一个要拆分的存储桶的指针
- 在任何通溢出的时候，在指针位置拆分桶
- 溢出条件由实际条件决定
  - 空间利用
  - 溢出链平均长度

![linear-hash1](CMU-DBMS-COURSE-06-NOTES/linear-hash1.png)

![linear-hash2](CMU-DBMS-COURSE-06-NOTES/linear-hash2.png)

![linear-hash3](CMU-DBMS-COURSE-06-NOTES/linear-hash3.png)

![linear-hash4](CMU-DBMS-COURSE-06-NOTES/linear-hash4.png)

![linear-hash6](CMU-DBMS-COURSE-06-NOTES/linear-hash6.png)

![linear-hash7](CMU-DBMS-COURSE-06-NOTES/linear-hash7.png)

![linear-hash8](CMU-DBMS-COURSE-06-NOTES/linear-hash8.png)

![linear-hash9](CMU-DBMS-COURSE-06-NOTES/linear-hash9.png)

![linear-hash10](CMU-DBMS-COURSE-06-NOTES/linear-hash10.png)

