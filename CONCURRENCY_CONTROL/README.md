## Concurrency control


### Shared vs Exclusive lock

These were introduced to ensure consistency.

#### Exclusive lock

This is acquired when you are changing something in the row/column and no body should be able to connect/attempt to read/modify this row/column (for a variety of reasons, security reasons, concurrency reasons, etc.,)

When you are trying to acquire an exclusive lock, there must not be any shared locks acquired on that row.

#### Shared Lock

When you acquire a shared lock and read. Do not allow to modify it and fail everything else


### Dead locks

When resource r1 is waiting for r2 to release a lock and r2 is waiting for r1 to release a lock. Nothing is gonna move forward.

The last one to enter the deadlock is the one that fails. Check `Dead Locks` video in Database engineering course.

Most DBs detect dead locks.


### Two-phase Locking

```sql
postgres=# begin transaction;
BEGIN
postgres=# select * from seats where id = 14 for update;
EXCLUSIVE LOCK IS ACQUIRED AT THIS STAGE, ANY OTHER TRANSACTION CANNOT ATTEMPT TO READ THIS ROW UNTIL RELEASED
postgres=# update seats set isbooked = 1, name = 'Sukeesh';
UPADTE 1
postgres=# commit;
COMMIT
```

### Solving the Double Booking Problem

use `for update` at the end to block.

Watch Database engineering course, 

[LEARN NICELY BY BUILDING A PROJECT]


**Way 2**

```sql
postgres=#begin;

postgres=# update seats set isbooked = 1, u = 'Sukeesh' where id = 1 and isbooked = 0;

UPDATE 1

postgres=#commit;
```

`and isbooked = 1` at the end of the query uses a lock (implicitly defined). Even works when uncomitted, other transaction can detect the lock on this row.


### SQL Pagination with offset is very slow.

```sql
# select * from table offset 100 limit 10
```

excludes 100 and gets next 10. This can cause to read rows repeatedly.

This offset addition is equal to pulling `offset + limit` rows from DB. It's super slow.

Use timestamp as solution (as you already know).


### Database Connection Pooling

For every request, no need to create a client, query and close the client. Instead, have a pool of connections, use the available connections from the pool. This is way faster as it saves create and destroy of client for every request.


(You can also lock a client in the pool for you, READ LATER)


