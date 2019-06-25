---
title: CMU-DBMS-COURSE-16-NOTES
date: 2019-06-19 23:47:35
tags: Concurrency control theory 
---

##### Semester(学期) status 

A DBMS's concurrency control and recovery components permeate(渗透) throughout the design of its entire architecture.

![Status](CMU-DBMS-COURSE-16-NOTES/Status.png)

##### Motivation

We both change the same record in a table at the same time.

How to avoid race(竞争) condition? (Lost Updates Concurrency Control)

You transfer $100 between bank accounts but there is a power failure.

What is the correct database state? (Durability Recovery)

##### Concurrency control & recovery

Valuable properties of DBMSs (DBMS的重要特性)

Based on concept of transactions with ACID properties.

##### Transactions

A transaction is the execution of a sequence of one or more operations (e.g., SQL queries) on a shared database to perform some higher-level function.

It is the basic unit of change in DBMS

**Example**

Move $100 from Andy' bank account to his bookie's account.

Transaction:

- Check whether Andy has $100.

- Deduct $100 from his account.

- Add $100 to his bookie's account.

##### Strawman System

Execute each txn one-by-one(i.e., serial order) as they arrive at the DBMS.

- One and only one txn can be running at the same time in the DBMS.

Before a txn starts, copy the entire database to a new file and make all changes to that file.

- If the txn completes successfully, overwrite the original file with the new one.
- If the txn fails, just remove the dirty copy.

##### Definitions

A txn may carry out many operations on the data retrieved from the database

However, the DBMS is only concerned about what data is read/written from/to the database.

- Changes to the "outside world" are beyond the scope of the DBMS.

##### Formal Definitions

Database: A fixed set of named data objects(e.g., A,B,C,...)

- We do not need to define what these objects are now

Transaction: A sequence of read and write operations(R(A), W(B), ...)

- DBMS's abstract view of a user program

##### Transactions in SQL

A new txn starts with the BEGIN command.

The txn stops with either COMMIT or ABORT:

- If commit, all changes are saved.
- If abort, all changes are undone so that it's like as if the txn never executed at all.
- Abort can be either self-inflicted or caused by the DBMS.

##### Correnctness criteria: ACID

**Atomicity:** All actions in the txn happen, or none happen.

**Consistency:** If each txn is consistent and the DB starts consistent, then it ends up consistent.

**Isolation:** Execution of one txn is isolated from that of other txns.

**Durability:** If a txn commits, its effects persist.

##### Atomicity of transactions

Two possible outcomes of executing a txn:

- Commit after completing all its actions.
- Abort(or be aborted by the DBMS) after executing some actions.

DBMS guarantees that txns are atomic.

- From user's point of view: txn always either executes all its actions, or executes no actions at all.

We take $100 out of Andy's account but then there is a power failure before we transfer it to his bookie.

*When the database comes back on-line, what should be the correct state of Andy's account?*

##### Mechanisms(机制) for ensuring atomicity

**Approach #1: Logging**

- DBMS logs all actions so that it can undo the actions of aborted transactions.
- Think of the like the black box in airplanes...

Logging used by all modern systems.

- Audit Trail(审计跟踪) & Efficiency Reasons

**Approach #2: Shadow Paging**

- DBMS makes copies of pages and txns make changes to those copies. Only when the txn commits it the page made visible to others.
- Originally from System R.

Few systems do this:

- CouchDB
- LMDB(OpenLDAP)

##### Consistency

The "world" represented by the database is logically correct. All questions asked about the data are given logically correct answers.

##### Database consistency

The database accurately models the real world and follows integrity(完整性) constraints.

Transactions in the future see the effects of transactions committed in the past inside of the database.

##### Transaction consistency

If the database is consistent before the transaction starts(running alone), it will also be consistent after.

Transaction consistency is the application's responsibility.

