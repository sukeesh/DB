## Transaction

It is a collection of queries. One unit of work. 


#### Transaction lifespan

Always begins with `BEGIN`. Ends with `COMMIT` to persist changes to disk. Sometimes things dont go right, we can `ROLLBACK`.

#### Nature of transaction

Usually transactions are used to change and modify a data. 

However, it is normal to have read only transaction. Example, you want to generate a report and you want to get consistent snapshot based **at the time of transaction**.


## Atomicity

- All queries in a transaction must succeed. If one query fails, all prior successful queries in the transaction should rollback. Or If DB went down prior to a commit of a transaction, all successful queries in the transactions should rollback. 


 

Due to many connections trying to change the same DB. Can one inflight transaction see changes mande by other transactions? This is called **Read Phenomena**.


### Read phenomena

#### Dirty reads

Inflight transaction reading change made by other transaction which is not comitted yet (not fully flushed).

#### Non-repeatable reads

Sales table
PID | QNT | PRICE
1   | 10  | $5
2   | 20  | $4

- Begin Tx1, `SELECT PID, QNT*PRICE FROM SALES` and `SELECT SUM(QNT*PRICE) FROM SALES`

For Query 1 in this Tx1,

1, 50
2, 80

In the middle of Tx1, Tx2 starts which is `UPDATE SALES SET QNT=QNT+5 WHERE PID=1` and `COMMIT TX2`

TX1, Query 2 starts to execute and we get `$155` when it should be `$130` we did read a committed value, but it gave us inconsistent results.


- PostgreSQL maintains a version of each row, so the corresponding row is read.
- MySQL and Oracle doesn't maintain any versions but they have undo tables but very expensive to read.

#### Phantom reads


Sales table
PID | QNT | PRICE
1   | 10  | $5
2   | 20  | $4

- Begin Tx1, `SELECT PID, QNT*PRICE FROM SALES` and `SELECT SUM(QNT*PRICE) FROM SALES`

For Query 1 in this Tx1,

1, 50
2, 80

In the middle of Tx1, Tx2 starts which is `INSERT INTO SALES VALUES(3, 10, 1)` and `COMMIT TX2`

TX1, Query 2 starts to execute and we get `$140` when it should be `$130` we did read a committed value that showed up in our range query.


#### Lost updates

Sales table
PID | QNT | PRICE
1   | 10  | $5
2   | 20  | $4


Tx1, Tx2 both start at the same time.
- Tx1 - `UPDATE SALES SET QNT = QNT + 10 WHERE PID = 1`
- Tx2 - `UPDATE SALES SET QNT = QNT + 5 WHERE PID = 1` Assuming Tx2 is committed before Tx1 is done, our update was overwritten by another transaction and as a result "lost".

### Isolation levels

Isolation levels for inflight transactions. These were invented to solve "Read phenomena".

`SET ISOLATION LEVEL`

#### Read uncommitted

No Isolation, any change from outside is visible to the transaction, committed or not.

#### Read committed

Most populor. Each txn only sees committed changes by other txns.

(Default Isolation levels for many DBs)

#### Repeatable read

To make read repeatable. The txn will make sure that when a query reads a row, that row will remain unchanged the transaction while its running. Each transaction only reads the data that was snapshotted before the start of this current txn.


#### Serializable

Transactions are run as if they are serialized one after the other. No concurrency.

**Note** Each DBMS implements isolation level differently.




- Pessimistic - Row level locks, table locks, page locks to avoid lost updates. (Don't touch this row)
- Optimistic - No locks, just track if things changed and fail the transaction if so.
- Repeatable read "locks" the rows it reads but it could be expensive if you read a lot of rows, postgres implements RR as snapshot. That is why you dont get phantom reads with postgres in repeatable read.
- Serializable are usually implemented with optimistic concurrency control, you can implement it pessimistically with SELECT FOR UPDATE.


## Consistency

#### Consistency in Data

- Referential Integrity

#### Consistency in Read

- Eventual consistency


## Durability

