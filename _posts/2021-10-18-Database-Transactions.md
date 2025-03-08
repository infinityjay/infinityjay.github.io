---
title:  Database transactions
categories:
  - Database
tags:
  - Transaction levels
  - Learning notes
---
Content

{% include toc %}

# Transaction Concept

A database transaction can be defined as a set of operations allowed by one or more databases. This set needs to support ACID features.

## ACID

### Atomicity

Atomicity means that all operations contained in a transaction are either all successful or all failed and rolled back. Therefore, if the operation of the transaction is successful, it must be fully applied to the database. If the operation fails, it cannot have any impact on the database.

### Consistency

Consistency means that the transaction must change the database from one consistent state to another, that is, a transaction must be in a consistent state before and after execution.

### Isolation

Isolation means that when multiple users access the database concurrently, such as when operating the same table, the transaction opened by the database for each user cannot be interfered with by the operations of other transactions, and multiple concurrent transactions must be isolated from each other.

Regarding the isolation of transactions, the database provides multiple isolation levels, which will be introduced later.

### Durability

Persistence means that once a transaction is committed, the changes to the data in the database are permanent, and the transaction commit operation will not be lost even if the database system encounters a failure. After the transaction method is committed, the user is prompted that the transaction operation is completed. When our program is executed and the prompt is seen, it can be determined that the transaction has been correctly committed. Even if there is a problem with the database at this time, our transaction must be fully executed. Otherwise, it will cause a major error that we see the prompt that the transaction is completed, but the database does not execute the transaction due to a failure.

# Common Problems in Transactions

## Dirty Read

* Dirty read means: reading data from another uncommitted transaction during the processing of a transaction.

When a transaction is modifying a certain data multiple times, and the multiple modifications in this transaction have not been committed, a concurrent transaction accesses the data, which will cause the data obtained by the two transactions to be inconsistent.

## Non-repeatable Read

* Non-repeatable read means: multiple queries within a transaction range return different data values. This is because another transaction is executed and submitted to modify a certain data value during the execution of this transaction, resulting in inconsistent results for the two queries of the transaction that has not yet ended.

<font color = red>The difference between non-repeatable read and dirty read is that dirty read is a transaction reading dirty data that has not been committed by another transaction, while non-repeatable read is a transaction reading data that has been committed by another transaction. </font>

## Phantom read

* Phantom read means that when a transaction queries the same range twice, the latter query sees rows that the former query did not see.

If snapshot reads are used in transactions, phantom reads will not occur, but phantom reads will occur if snapshot reads and current reads are mixed. For more information about snapshot reads and current reads, see [Database Snapshots (SQL Server)](https://docs.microsoft.com/zh-cn/sql/relational-databases/databases/database-snapshots-sql-server?view=sql-server-ver15).

<font color = red>Both phantom reads and non-repeatable reads read another committed transaction (dirty reads read uncommitted transactions). The difference is that non-repeatable reads query the same data item, but phantom reads target a batch of data as a whole (such as the number of data)</font>.

## Summary

<font color = red>"Dirty read", "non-repeatable read" and "phantom read" are actually database read consistency problems,</font> which must be solved by the database providing a certain transaction isolation mechanism.

<font color = red>The methods for implementing transaction isolation in the database can be divided into the following two types</font>:

1. Lock before reading data to prevent other data from modifying the data;

2. Without locking, a data snapshot consistent with the data request time point is generated through a certain mechanism, and this snapshot is used to provide a certain level of consistent reading. From the user's perspective, the database provides different versions of the same data, so this method is also called data multi-version concurrency control.

# Transaction isolation level

In order to resolve the contradiction between "isolation" and "concurrency", ISO/ANSI SQL92 defines 4 transaction isolation levels, each with different isolation levels and different side effects allowed. Applications can balance the contradiction between "isolation" and "concurrency" by selecting different isolation levels according to their own business logic requirements.

## There are four types of transaction isolation levels (in descending order):

### 1. Serializable

This is the highest isolation level of the database. At this level, transactions are "serialized and executed sequentially", that is, they are queued and executed one by one.

At this level, "dirty reads", "non-repeatable reads" and "phantom reads" can all be avoided, but the execution efficiency is extremely poor and the performance overhead is also the largest, so basically no one will use it.

### 2. Repeatable Read

Repeatable read, as the name suggests, is an isolation level specifically designed for the situation of "non-repeatable reads". Naturally, it can effectively avoid "non-repeatable reads". <font color = red>And it is also the default isolation level of MySQL</font>.

At this level, ordinary queries also use "snapshot reads", but unlike "read committed", when a transaction is started, "updates" are not allowed, and "non-repeatable reads" are precisely because data is modified between two reads. Therefore, "repeatable reads" can effectively avoid "non-repeatable reads", but cannot avoid "phantom reads" because phantom reads are caused by "insert or delete operations".

### 3. Read Committed

Read committed, as the name suggests, can only read the content that has been committed. <font color = red>This is the most commonly used isolation level in various systems, and it is also the default isolation level for SQL Server and Oracle</font>, which ensures that a transaction will not read data that has been modified but not committed by another parallel transaction, avoiding "dirty reads", but cannot avoid "phantom reads" and "non-repeatable reads". This level is suitable for most systems. Here is a little more: Why is "read committed" the same as "read uncommitted", without query locking, but can avoid dirty reads?

This brings us to another mechanism, "snapshot", and this kind of read that can ensure consistency and not lock is also called "snapshot read".

Assuming there is no "snapshot read", when an updated transaction is not committed, another transaction that queries the updated data will be blocked because it cannot query. In this case, the concurrency capability is quite poor. "Snapshot read" can complete high-concurrency queries, but "read commit" can only avoid "dirty reads" and cannot avoid "non-repeatable reads" and "phantom reads".

### 4. Read Uncommitted

Read uncommitted, as the name suggests, means that uncommitted content can be read. Therefore, at this isolation level, the query will not be locked. Because the query is not locked, the consistency of this isolation level is the worst, and "dirty reads", "non-repeatable reads" and "phantom reads" may occur. Unless there are special circumstances, this isolation level is basically not used.

## Summary

Whether these four transaction isolation levels will cause the above three problems is summarized as follows:

| Transaction isolation level | Dirty read | Non-repeatable read | Phantom read |
| :------------------------------- | :--: | :--------: | :--: |
| Serializable (S) | No | No | No |
| Repeatable read (RR) | No | No | Yes |
| Read committed (RC) | No | Yes | Yes |
| Read uncommitted (RU) | Yes | Yes | Yes |

The highest of the above four isolation levels is the Serializable level, and the lowest is the Read uncommitted level. Of course, the higher the isolation level, the more it can ensure the integrity and consistency of the data, but the lower the execution efficiency and the greater the impact on concurrent performance. For example, a level like Serializable uses a lock table (similar to the lock in Java multithreading) to make other threads wait outside the lock, so the isolation level you choose should be based on the actual situation.

