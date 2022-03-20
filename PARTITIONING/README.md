### What is Partitioning?

Split a large table into multiple tables and query only part of the bigger table.
For eg,

Split `CUSTOMERS` table with 1 million rows into 5 tables. `CUSTOMER_200K`, `CUSTOMER_400K`, `CUSTOMER_600K`, `CUSTOMER_800K` and `CUSTOMER_1M`.

### Vertical vs Horizontal Partitioning

- Horizontal Partitioning splits rows into partitions
	- Range or list
- Vertical partitioning splits columns partitions
	- Large column (blob) that you can store in a slow access drive in its own tablespace.

### Partitioning Types

- By Range
	- Dates, ids (e.g. by logdate or customerid from to)
- By List
	- Discrete values (e.g. states TN, TS, AP, etc.) or zip codes.
- By Hash
	- Hash functions (consistent hashing)

### Horizontal Partitioning vs Sharding

- HP splits big table into multiple tables in the same database, client is agnostic.
- Sharding splits big table into multiple tables across multiple database servers.
- HP table name changes (or schema)
- Sharding everything is the same but server changes.

### How to partition in Postgres?

```sql
postgres=# create table grades_parts (id serial not nullm g int not null) partition by range(g);
```
Now, above specifies that we are partitioning but partion tables should be created by ourselves.

```sql
postgres=# create table g3560 (like grades_parts including indexes);
CREATE TABLE
postgres=# create table g6080 (like grades_parts including indexes);
CREATE TABLE
```

Now, attach partitions one-by-one to the major table.

```sql
postgres=# alter table grades_parts attach partition g0035 for values from (0) to (35);
ALTER TABLE
postgres=# alter table grades_parts attach partition g3560 for values from (35) to (60);
ALTER TABLE
```

What happens while inserting?

```sql
postgres=# insert into grades_parts select * from grades_org;
```
While inserting above, it decides which partition to insert into.

What happens when an index is created on leader/master table?

```sql
postgres=# create index grades_parts_idx on grades_parts(g);
```
It is gonna create index on partitions table as well.

Read about `set enable_partitioning_pruning = off;` <- Why does this even exist? This will make partitions indexes useless.


### Pros of Partitioning

- Improves query performance when accessing a single partition.
- Sequential scan vs Scattered index scan.
- Easy bulk loading (attach partition)
- Archive old data that are barely accessed into cheap storage.

### Cons of Partitioning

- Updates that move rows from a partition to another (slow or fail sometimes)
- Inefficient queries could accidently scan all partitions resulting in slower performance
- Schema changes can be challenging (DBMS could manage it though)



