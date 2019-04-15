---
title: CMU-DBMS-COURSE-09-NOTES
date: 2019-04-13 14:53:23
tags:
---

- 我们假设到目前为止我们讨论的所有数据结构都是单线程的
- 但是我们需要允许多个线程安全地访问我们的数据结构，以利用额外的CPU核心

**并发控制**

- 并发控制协议是DBMS用来确保共享对象上并发操作产生“正确”结果的方法
- 协议的正确性标准可能有所不同
  - 逻辑正确性：我能看到我应该看到的数据吗？
  - 物理正确性：对象的内部表示是否正确？

**锁 vs 闩**

- 锁
  - 保护索引的逻辑内容不受其他线程的影响
  - 确保线程的持久化操作
  - 需要有能力回滚变化
- 闩
  - 保护索引内部数据结构关键部分不受其他线程的影响
  - 确保操作的持久化
  - 不需要保证能够回滚变化

|        | Locks(锁)              | Latches(闩)        |
| ------ | ---------------------- | ------------------ |
| 区分   | 用户事务               | 线程               |
| 保护   | 数据库的内容           | 在内存中的数据结构 |
| 期间   | 整个事务               | 关键部分           |
| 模式   | 共享、独占、更新、意图 | 读、写             |
| 死锁   | 检测 & 分辨            | 避开               |
| 通过   | 等待，超时，中断       | 专业的编码         |
| 保存在 | 锁管理器               | 被保护的数据结构中 |

**闩锁模式**

- 读模式
  - 允许多个线程在同一时刻读取同一个项目
  - 一个线程可以获得读闩锁，当另一个线程也在读模式
- 写模式
  - 只有一个线程允许访问项目
  - 当宁一个线程获得一把任意模式的闩锁时，一个线程不能获得一把写闩锁

**B+树并发控制**

- 我们希望允许多个线程同时读和更新B+树索引
- 我们需要避免两类问题
  - 试图同时修改节点内容的多个线程
  - 一个线程遍历树，另一个线程拆分/合并解节点

![muti-thr1](CMU-DBMS-COURSE-09-NOTES/muti-thr1.png)

![muti-thr2](CMU-DBMS-COURSE-09-NOTES/muti-thr2.png)

![muti-thr3](CMU-DBMS-COURSE-09-NOTES/muti-thr3.png)

![muti-thr4](CMU-DBMS-COURSE-09-NOTES/muti-thr4.png)

![muti-thr5](CMU-DBMS-COURSE-09-NOTES/muti-thr5.png)

![muti-thr6](CMU-DBMS-COURSE-09-NOTES/muti-thr6.png)

![muti-thr7](CMU-DBMS-COURSE-09-NOTES/muti-thr7.png)

- 协议允许多线程同时访问和修改B+树
- 基本的idea
  - 为父节点获取闩锁
  - 为子节点获取闩锁
  - 如果父节点安全，释放闩锁
- 一个安全的节点是更新时不会拆分或合并的节点
  - 没有满（插入操作的时候）
  - 超过一半满（删除操作的时候）
- 搜索：从根开始向下，反复
  - 为子节点获取读锁
  - 然后释放父节点的读锁
- 插入/删除：从根部开始向下，根据需要获得读锁。一旦子节点被锁上，检查它是否安全
  - 如果子节点安全，释放祖先节点多有的锁

![latch1](CMU-DBMS-COURSE-09-NOTES/latch1.png)

![latch2](CMU-DBMS-COURSE-09-NOTES/latch2.png)

![latch3](CMU-DBMS-COURSE-09-NOTES/latch3.png)

![latch4](CMU-DBMS-COURSE-09-NOTES/latch4.png)

![latch5](CMU-DBMS-COURSE-09-NOTES/latch5.png)

![latch6](CMU-DBMS-COURSE-09-NOTES/latch6.png)

![latch7](CMU-DBMS-COURSE-09-NOTES/latch7.png)

![latch8](CMU-DBMS-COURSE-09-NOTES/latch8.png)![latch9](CMU-DBMS-COURSE-09-NOTES/latch9.png)

![latch10](CMU-DBMS-COURSE-09-NOTES/latch10.png)

![latch11](CMU-DBMS-COURSE-09-NOTES/latch11.png)

![latch12](CMU-DBMS-COURSE-09-NOTES/latch12.png)

![latch13](CMU-DBMS-COURSE-09-NOTES/latch13.png)

![latch14](CMU-DBMS-COURSE-09-NOTES/latch14.png)

![latch15](CMU-DBMS-COURSE-09-NOTES/latch15.png)

![latch16](CMU-DBMS-COURSE-09-NOTES/latch16.png)

![latch17](CMU-DBMS-COURSE-09-NOTES/latch17.png)

![latch18](CMU-DBMS-COURSE-09-NOTES/latch18.png)

![latch19](CMU-DBMS-COURSE-09-NOTES/latch19.png)

![latch20](CMU-DBMS-COURSE-09-NOTES/latch20.png)

![latch21](CMU-DBMS-COURSE-09-NOTES/latch21.png)

![latch22](CMU-DBMS-COURSE-09-NOTES/latch22.png)![latch23](CMU-DBMS-COURSE-09-NOTES/latch23.png)



![latch24](CMU-DBMS-COURSE-09-NOTES/latch24.png)

![latch25](CMU-DBMS-COURSE-09-NOTES/latch25.png)

**思考**

- 下面这个示例，哪个是第一个针对B+树要执行的更新操作?

![observation1](CMU-DBMS-COURSE-09-NOTES/observation1.png)

- 每次在根上执行的写锁操作都会成为高并发的性能瓶颈

**更好的加锁算法**

- 假设叶节点是安全的
- 使用读闩锁抵达到它，并验证它是否安全
- 如果叶节点不安全，用读锁执行先前的算法

![better1](CMU-DBMS-COURSE-09-NOTES/better1.png)

![better2](CMU-DBMS-COURSE-09-NOTES/better2.png)

![better3](CMU-DBMS-COURSE-09-NOTES/better3.png)

![better4](CMU-DBMS-COURSE-09-NOTES/better4.png)

![better5](CMU-DBMS-COURSE-09-NOTES/better5.png)

![better6](CMU-DBMS-COURSE-09-NOTES/better6.png)

![better7](CMU-DBMS-COURSE-09-NOTES/better7.png)

![better8](CMU-DBMS-COURSE-09-NOTES/better8.png)

- 搜索：和前面的算法一样
- 插入/删除
  - 像搜索一样设置闩锁，抵达叶节点，并在叶节点上设置写闩锁
  - 如果叶节点的操作不安全，释放所有闩锁，然后使用具有写闩锁的插入/删除的协议重新启动线程

- 这种方法乐观的假定只修改叶节点；否则，在第一次传递到叶节点时设置的读锁是浪费的

- 到目前为止，所有示例中的线程都以“从上而下”的方式获得了锁存
  - 线程只能从低于其当前节点的节点获取闩锁
  - 如果所需的闩锁不可用，则线程必须等到它可用为止
  - 但是如果我们想从一个叶节点移动到另一个叶节点呢

**叶节点扫描**

![leaf-node-scan1](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan1.png)

![leaf-node-scan2](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan2.png)

![leaf-node-scan3](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan3.png)

![leaf-node-scan4](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan4.png)

![leaf-node-scan5](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan5.png)

![leaf-node-scan6](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan6.png)

![leaf-node-scan7](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan7.png)

![leaf-node-scan8](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan8.png)

![leaf-node-scan9](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan9.png)

![leaf-node-scan10](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan10.png)

![leaf-node-scan11](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan11.png)

![leaf-node-scan12](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan12.png)

![leaf-node-scan13](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan13.png)

![leaf-node-scan14](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan14.png)

![leaf-node-scan15](CMU-DBMS-COURSE-09-NOTES/leaf-node-scan15.png)

- 闩锁不支持死锁检测和避免，我们唯一能解决这个问题的方法是通过编码规则
- 叶节点同级闩锁采集协议必须支持“无等待”模式，B+树代码必须处理失败的锁存采集

**延迟父节点更新**

- 每当一个叶子节点溢出时，我们必须至少更新三个节点
  - 正在拆分的叶节点
  - 新创建的叶节点
  - 父节点
- B link - 树优化：当一个叶节点溢出时，延迟父节点的更新操作

![delay-parent1](CMU-DBMS-COURSE-09-NOTES/delay-parent1.png)

![delay-parent2](CMU-DBMS-COURSE-09-NOTES/delay-parent2.png)

![delay-parent3](CMU-DBMS-COURSE-09-NOTES/delay-parent3.png)

![delay-parent4](CMU-DBMS-COURSE-09-NOTES/delay-parent4.png)

![delay-parent5](CMU-DBMS-COURSE-09-NOTES/delay-parent5.png)

![delay-parent6](CMU-DBMS-COURSE-09-NOTES/delay-parent6.png)

![delay-parent7](CMU-DBMS-COURSE-09-NOTES/delay-parent7.png)

![delay-parent8](CMU-DBMS-COURSE-09-NOTES/delay-parent8.png)

![delay-parent9](CMU-DBMS-COURSE-09-NOTES/delay-parent9.png)

![delay-parent10](CMU-DBMS-COURSE-09-NOTES/delay-parent10.png)

**结论**

- 使一个数据结构线程安全似乎很容易理解，但在实践中缺非常困难
- 我们关注的是B+树，但同样高级技术叶适用于其他的数据结构

