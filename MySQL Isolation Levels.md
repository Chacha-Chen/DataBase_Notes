# MySQL Isolation Levels
- READ_UNCOMMITTED
- READ_COMMITTED
- REPEATABLE_READ (default)
- SERIALIZABLE

InnoDB supports each of the transaction isolation levels described here using <mark>different locking strategies</mark>.

## Situation
REPEATABLE_READ (default)  

SERIALIZABLE: enforces even stricter rules, and is used mainly in specialized situations, such as with XA transactions and for troubleshooting issues with concurrency and deadlocks.

READ\_COMMITTED & READ_UNCOMMITTED: in situations such as bulk reporting where precise consistency and repeatable results are less important than <mark>minimizing the amount of overhead for locking</mark>. 

## REPEATABLE_READ
snapshot  
A representation of data at a particular time。 which remains the same even as changes are committed by other transactions. Used by certain isolation levels to <mark>allow consistent reads</mark>.

### consistent non-locking read:
- plain SELECT statements
 
同一个transaction内的读操作是一致的，在第一次select操作时生成了一个现有表的snapshot，之后的读都从这个snapshot中读。

### locking reads
- SELECT with FOR UPDATE or LOCK IN SHARE MODE
- UPDATE
- DELETE

For a unique index with a unique search condition, InnoDB locks only the index record found, not the gap before it.

For other search conditions, InnoDB locks the _index range scanned_, using gap locks or next-key locks to block insertions by other sessions into the gaps covered by the range. 

##  READ COMMITTED

Each consistent read, even within the same transaction, sets and reads its own fresh snapshot. (但注意snapshot只会在修改提交后在取得新的snapshot）

### For locking reads 
- SELECT with FOR UPDATE or LOCK IN SHARE MODE
- UPDATE statements
- DELETE statements  
InnoDB locks only index records, not the gaps before them, and thus permits the free insertion of new records next to locked records. Gap locking is only used for foreign-key constraint checking and duplicate-key checking.

一般来说，gap locking is disabled，所以可能有phantom幻影现象，即同一事务第二次读出现新刚insert的row。
 
For UPDATE or DELETE statements, InnoDB holds locks only for rows that it updates or deletes. Record locks for nonmatching rows are released after MySQL has evaluated the WHERE condition. This greatly reduces the probability of deadlocks, but they can still happen.

For UPDATE statements, if a row is already locked, InnoDB performs a “semi-consistent” read, returning the latest committed version to MySQL so that MySQL can determine whether the row matches the WHERE condition of the UPDATE. If the row matches (must be updated), MySQL reads the row again and this time InnoDB either locks it or waits for a lock on it.

## READ UNCOMMITTED

SELECT statements are performed in a nonlocking fashion, but a possible earlier version of a row might be used. Thus, using this isolation level, such reads are not consistent. This is also called a dirty read. Otherwise, this isolation level works like READ COMMITTED.

## SERIALIZABLE

This level is like REPEATABLE READ, but InnoDB implicitly <mark>converts all plain SELECT statements to SELECT ... LOCK IN SHARE MODE if autocommit is disabled</mark>. If autocommit is enabled, the SELECT is its own transaction. It therefore is known to be read only and can be serialized if performed as a consistent (nonlocking) read and need not block for other transactions. (To force a plain SELECT to block if other transactions have modified the selected rows, disable autocommit.)
 
 
#Consistent Nonlocking Reads

A consistent read means that InnoDB uses <mark>multi-versioning</mark> to present to a query a snapshot of the database at a point in time. **The query sees the changes made by transactions that committed before that point of time, and no changes made by later or uncommitted transactions.** <mark>The exception to this rule is that the query sees the changes made by earlier statements **within the same transaction**.</mark> This exception causes the following anomaly: If you update some rows in a table, a SELECT sees the latest version of the updated rows, but it might also see older versions of any rows. If other sessions simultaneously update the same table, the anomaly means that you might see the table in a state that never existed in the database.

The exception to this rule is that the query sees the changes made by earlier statements within the same transaction. 

- REPEATABLE READ (the default level), all consistent reads within the same transaction read the snapshot established by the first such read in that transaction. You can get a fresher snapshot for your queries by committing the current transaction and after that issuing new queries.

- READ COMMITTED isolation level, each consistent read within a transaction sets and reads its own fresh snapshot.

<mark>Consistent read is the default mode in which InnoDB processes SELECT statements in READ COMMITTED and REPEATABLE READ isolation levels. A consistent read does not set any locks on the tables it accesses, and therefore other sessions are free to modify those tables at the same time a consistent read is being performed on the table.</mark>

 A consistent read does not set any locks on the tables it accesses, and therefore other sessions are free to modify those tables at the same time a consistent read is being performed on the table.

InnoDB gives your transaction a timepoint according to which your query sees the database. If another transaction deletes a row and commits after your timepoint was assigned, you do not see the row as having been deleted. Inserts and updates are treated similarly.

an ordinary SELECT statement->issue a consistent read mode->no locks on the tables->other sessions are free to modify（but the table you see is from the transaction timepoint, later changes cannot be seen)


所以，READ COMMITTED&REPEATABLE READ，
