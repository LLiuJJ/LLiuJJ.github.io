---
title: CMU-DBMS-COURSE-18-NOTES (Timestamp Ordering Concurrency Control - Optimistic)
date: 2019-10-10 20:21:07
tags: Timestamp Ordering Concurrency Control

---

**1 Timestamp Ordering Concurrency Control**

Timestamp ordering(T/O) is a optimistic class of concurrency control protocols where the DBMS assumes that transaction conflicts are rare. Instead of requiring transactions to acquire locks before thet are allowed to read/write to a database object, the DBMS instead uses timestamps to determine the serializability order of transactions.

Each transaction T(i) is assigned a unique fixed timestamp that is monotonically(单调的) increasing:
- Let TS(Ti) be the timestamp allocated to transaction Ti
- Different schemes assign timestamps at different times during the transaction

If TS(Ti) < TS(Tj), then the DBMS must ensure that the execution schedule is equivalent to a serial schedule where Ti appears before Tj.

Muliple timestamp allocation implementation strategies(策略).
- System clock
- Logical counter
- Hybrid

**2 Basic Timestamp Ordering (BASIC T/O)**
Every database object X is tagged with timestamp of the last transaction successfully did read/write:
- W-TS(X): Write timestamp on object X.
- R-TS(X): Read timestamp on object X.

The DBMS check timestamps for every operation. If transaction tries to access an object "from the future", then the DBMS aborts that transaction and restart it.

Read Operation:
- IF TS(Ti) < W-TS(X) this violation timestamp order of Ti with regard to the writer of X. Thus you abort Ti and restart it with same TS.
- Else:
  - Allow Ti to read X.
  - Update R-TS(X) to max(R-TS(X), TS(Ti)).
  - Have to make a local copy of X to ensure repeatable reads for Ti.
  - Last step may be skipped in lower isolation levels.

Write Operation:
- If TS(Ti) < R-TS(X) or TS(Ti) < W-TS(X), abort and restart Ti.
- Else:
  - Allow Ti to write X and update W-TS(X) to Ti.
  - Also have to make a local copy of X to ensure repeatable reads for Ti.

Optimization: Thomas Write Rule
- If TS(Ti) < R-TS(X): Abort and restart Ti
- If TS(Ti) < W-TS(X):
  - Thomas Write Rule: Ignore the write and allow transaction to continue.
  - Note that this violates timestamp order of Ti but is okay because no other transaction will read Ti's write to object X.
- Else: Allow Ti to write X and update W-TS(X)

The Basic T/O protocol generates a schedule that is conflict serializable if you do not use Thomas Write Rule. It cannot have deadlocks because no transation ever waits. But there is a possibility of starvation for log transactions if short transactions keep causing conflicts.

It also permits schedules that are not recoverable. A schedule is recoverable if transactions commit only after all transactions whose changes they read or commit. Otherwise, the DBMS cannot guarantee that transactions read data that will be restored after recovering from a crash.

Potential(潜在) Issues:
- High overhead(开销) from copying data to transation's workspace and from updating timestamp.
- Long running transactions can get starved: The likelihood that a transaction will read something from a newer transaction increases.
- Suffers from the timestamp allocation bottleneck on highly concurrent systems.

