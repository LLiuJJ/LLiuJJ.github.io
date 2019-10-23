---
title: CMU-DBMS-COURSE-17-NOTES (Two Phase Locking - Pessimistic)
date: 2019-08-04 22:49:45
tags: Two Phase Locking
---

##### Example![ex1](CMU-DBMS-COURSE-17-NOTES/ex1.png)

We need a way to guarantee that all execution schedules are correct(i.e, serializable) without knowing the entire schedule adead of time.

Solution: Use locks to protect database objects.

##### Executing with locks

![LOCK-1](CMU-DBMS-COURSE-17-NOTES/LOCK-1.png)

![LOCK-2](CMU-DBMS-COURSE-17-NOTES/LOCK-2.png)

![LOCK-3](CMU-DBMS-COURSE-17-NOTES/LOCK-3.png)

![LOCK-4](CMU-DBMS-COURSE-17-NOTES/LOCK-4.png)

|                 | Locks                                | Latches                   |
| --------------- | ------------------------------------ | ------------------------- |
| **Separate...** | User transactions                    | Threads                   |
| **Protect...**  | Database Contents                    | In-Memory Data Structures |
| **During...**   | Entire Transactions                  | (关键的)ritical Sections  |
| **Modes...**    | Shared, Exclusive, Update, Intention | Read, Write               |
| Deadlock        | Detection & Resolution               | Avoidance(避免)           |
| **…by...**      | Waits-for, Timeout, Aborts           | Coding Discipline(准则)   |
| Kept in ...     | Lock Manager                         | Protected Data Structure  |

 (注：闩锁和锁定既有相同点又有不同点。相同点在于它们都是用于实现串行化的资源。而不同点则在于闩锁（Latch）是一个低级别、轻量级的锁，获得和释放的速度很快，以类似于信号灯的方式实现。而锁定（Lock）则可能持续的时间很长，通过使用队列，按照先进先出的方式实现。Latch是非入队的，而Lock是入队的。Latch不存在死锁，而Lock中存在)

**Basic Lock types**

- S-LOCK: Shared locks for reads
- X-LOCK: Exclusive locks for writes

**Executing with locks**

Transactions request locks(or upgrades).

Lock Manager grants or blocks requests.

Transactions release locks.

Lock manager updates its internal lock-table.

- It keeps track of what transactions hold what locks and what transactions are waiting to acquire any locks.

![LOCK-EXE-1](CMU-DBMS-COURSE-17-NOTES/LOCK-EXE-1.png)

![LOCK-EXE-2](CMU-DBMS-COURSE-17-NOTES/LOCK-EXE-2.png)

![LOCK-EXE-3](CMU-DBMS-COURSE-17-NOTES/LOCK-EXE-3.png)

![LOCK-EXE-4](CMU-DBMS-COURSE-17-NOTES/LOCK-EXE-4.png)

![LOCK-EXE-5](CMU-DBMS-COURSE-17-NOTES/LOCK-EXE-5.png)

**Concurrency contorl protocol**

Two-phase locking(2PL) is a concurrency control protocol that determines whether a txn is allowed to access an object in the database on the fly.

The protocol does not need to know all of the queries that a txn will execute ahead of time.

**Two phase locking**

- Phase #1: Growing

Each txn requests the locks that it needs from the DBMS's lock manager.

The lock manager grants/denies lock requests.

- Phase #2: Shrinking 

The txn is allowed to only release locks that it previously acquired. It cannot acquire new locks.

![GrowingPhase](CMU-DBMS-COURSE-17-NOTES/GrowingPhase.png)

The txn is not allowed to acquire/upgrade locks after the growing phase finishes.

![ShrinkingPhase](CMU-DBMS-COURSE-17-NOTES/ShrinkingPhase.png)

The txn is not allowed to acquire/upgrade locks after the growing phase finishes.

**Executing with 2PL**

![2PL-EXEC1](CMU-DBMS-COURSE-17-NOTES/2PL-EXEC1.jpg)

![2PL-EXEC2](CMU-DBMS-COURSE-17-NOTES/2PL-EXEC2.jpg)

![2PL-EXEC3](CMU-DBMS-COURSE-17-NOTES/2PL-EXEC3.jpg)

![2PL-EXEC4](CMU-DBMS-COURSE-17-NOTES/2PL-EXEC4.jpg)

2PL on its own is sufficient to guarantee conflict serializability.

It generates schedules whose precedence(优先) graph is acyclic(非循环的).

**2PL Cascading aborts**

![2PL-CASCADING-ABORT](CMU-DBMS-COURSE-17-NOTES/2PL-CASCADING-ABORT.jpg)

This is a permissible schedule in 2PL, but the DBMS has to also abort T2 when T1 aborts.

- Any information about T1 cannot be "leaked" to the outside world.

**Strict two-phase locking**

The txn is not allowed to acquire/upgrade locks after the growing phase finishes.

Allows only conflict serializable schedules, but it is often stronger than needed for some apps.

![STRICT-TWO-PHASE-LOCKING](CMU-DBMS-COURSE-17-NOTES/STRICT-TWO-PHASE-LOCKING.jpg)

A schedule is strict if a value written by a txn is not read or overwritten by other tens until that txn finishes.

Advantages:

- Does not incur cascading aborts.
- Aborted txns can be undone by just restoring(恢复) original values of modified tuples.

**Example**

T1 - Move $100 from Andy's account(A) to his bookie's account(B).

T2 - Compute the total amount in all accounts and return it to the application.

![2PL-EX1](CMU-DBMS-COURSE-17-NOTES/2PL-EX1.jpg)

- Non 2PL example

![NO-2PL-1](CMU-DBMS-COURSE-17-NOTES/no-2pl-1.png)

![NO-2PL-2](CMU-DBMS-COURSE-17-NOTES/no-2pl-2.png)

![NO-2PL-3](CMU-DBMS-COURSE-17-NOTES/no-2pl-3.png)

![NO-2PL-4](CMU-DBMS-COURSE-17-NOTES/no-2pl-4.png)

- 2PL example

![2PL-1](CMU-DBMS-COURSE-17-NOTES/2PL-1.png)

![2PL-2](CMU-DBMS-COURSE-17-NOTES/2PL-2.png)

![2PL-3](CMU-DBMS-COURSE-17-NOTES/2PL-3.png)


- Strict 2PL example

![STRICT 2PL 1](CMU-DBMS-COURSE-17-NOTES/STRICT-2PL-1.png)

![STRICT 2PL 2](CMU-DBMS-COURSE-17-NOTES/STRICT-2PL-2.png)

![STRICT 2PL 3](CMU-DBMS-COURSE-17-NOTES/STRICT-2PL-3.png)

- 2PL may lead to DeadLock

![DEADLOCK-1](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-1.png)

![DEADLOCK-2](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-2.png)

![DEADLOCK-3](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-3.png)

![DEADLOCK-4](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-4.png)

A deadlock is a cycle of transactions waiting for locks to be released by each other.

There is two ways of dealing with deadlocks
- 1.Deadlock Detection(检测)
- 2.Deadlock Prevention(预防)

**Deadlock Detection**

Model: The DBMS creates a waits-for graph to keep track of what locks each txn is waiting to aquire:
In follow Model
- Nodes are transactions
- Edge from T(i) to T(j) if T(i) is waiting for T(j) to release a lock.

The system will periodically check for cycles in waits-for graph and then make a decision on how to break it.

![DEADLOCK-DETECTION-1](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-DETECTION-1.png)

![DEADLOCK-DETECTION-2](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-DETECTION-2.png)

![DEADLOCK-DETECTION-3](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-DETECTION-3.png)

![DEADLOCK-DETECTION-4](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-DETECTION-4.png)

**deadlock handing**

When the DBMS detects a deadlock, it will select a "victim(牺牲者)" txn to rollback to break the cycle.

Selecting the proper victim depends on a lot of different variables...
- By age (lowest timestamp)
- By progress (least/most queries executed)
- By the items already locked
- By the txns that we have to rollback with it

We also should consider the times a txn has been restarted in the past to prevent starvation(防止饥饿).

The victim txn will either restart or abort(more common) depending on how it was invoked.

There is a trade-off(权衡) between the frequency of checking for deadlocks and how long txns have to wait before deadlocks are broken.

After selecting a victim txn to abort, the DBMS can also decide on how far to rollback the txn's changes.

- 1: Completely
- 2: Minimally

**Deadlock Prevention**

When a txn tries to acquire a lock that is held by another txn, the DBMS kills one of them to prevent a deadlock.

This approach does not require a waits-for graph or detection algorithm.

Assign priorities(分配优先级) based on timestamps:

- Older Timestamp = Higher priority(e.g., T(1) > T(2))

Wait-Die("Old Waits for Young")

- If reqesting txn has higher priority than holding txn, then requesting txn waits for holding txn.

- Otherwise requesting txn aborts.

Wound(受伤)-Wait("Young Waits for Old")

- If requesting txn has higher priority than holding txn, then holding txn aborts and releases lock.

- Otherwise requesting txn waits.

![DEADLOCK-PREVENTION](CMU-DBMS-COURSE-17-NOTES/DEADLOCK-PREVENTION.png)

**Lock Granularities(锁粒度)**

When we say that a txn acquires a "lock", what does that actually mean?
- On an Attribute? Tuple? Page? Table?

Ideally, each txn should obtain fewest number of locks that is needed

![LOCK-HIERARCHY1](CMU-DBMS-COURSE-17-NOTES/LOCK-HIERARCHY-1.png)

![LOCK-HIERARCHY2](CMU-DBMS-COURSE-17-NOTES/LOCK-HIERARCHY-2.png)

**Example**

What locks should follow txns obtain?
T(1) - get the balance of Andy's shady off-shore bank account(非法境外账户).
T(2) - Increase Lin's bank account balance by 1%.

- Exclusive(独占的) + Shared for leafs of lock tree.
- Special Intention locks(意向锁) for higher levels.

**Intention Lock**
An intention lock allows a higher level node to be locked in shared or exclusive mode without having to check all descendent(派生的) nodes.

If a node is in an intention mode, then explicit locking is being done at a lower level in the tree.

Intention-Shared(IS)
- Indicates explicit locking at a lower level with shared locks.

Intention-Exclusive(IX)
- Indicates locking at lower level with exclusive or shared locks.

Shared + Intention-Exclusive(SIX)
- The subtree rooted by that node is locked explicitly in shared mode and explicit locking is being done at a lower level with exclusive-mode locks.

