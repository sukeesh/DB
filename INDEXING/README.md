### Index

```
# explain analyze select * from table;
```

Uses B-TREE

#### Index only scan

If everything is on index.

#### Understanding the SQL Query Planner and Optimizer


Seq scan.

Sometimes DBs do parallel Seq scan with multiple workers.

```sql
postgres=# explain select * from temp;
                          QUERY PLAN
--------------------------------------------------------------
 Seq Scan on temp  (cost=0.00..14425.01 rows=1000001 width=4)
(1 row)
```

cost=0.00 -> Time taken to fetch the first page or in other words, time for startup.
cost=0.00..(14425.01) -> Estimated time

rows=1000001 -> Estimated number of rows

width=31 -> Sum of all the bytes of all the columns. Average row size of the result.


#### How to read?

```sql
postgres=# explain select * from temp order by t desc;
                             QUERY PLAN
--------------------------------------------------------------------
 Sort  (cost=127757.46..130257.46 rows=1000001 width=4)
   Sort Key: t DESC
   ->  Seq Scan on temp  (cost=0.00..14425.01 rows=1000001 width=4)
(3 rows)

```
Start from inner loop of the explain and move upwards.

Larger the width, larger the TCP packets, networking etc.,


### Index Scan vs Index Only Scan

- **Index Scan:** Use index to scan something else. You have to go back to table to fetch columns after we query index.
- **Index Only Scan:** Use index and fetch from index only. Check below for Nonkey.

```sql
postgres=# create index id_idx on grades(id) include (name)
```
In the above, `name` is called Nonkey. These are stored in the index itself.



```sql
postgres=# explain (analyze, buffer) select * from table;
```
`buffer` gives tells us about cache, shared hit.

`<Read more about buffer>`


```sql
postgres=# vacuum (verbose) table_name;
```

Remove dead rows from index, fixes, updates pages etc.,


### Bitmap index

Bitmap Indexing is a special type of database indexing that uses bitmaps. This technique is used for huge databases, when column is of low cardinality and these columns are most frequently used in the query. 

`Employee` table

| No      | NewEmp    |Job|
|---------|-----------|---|
| 1       | Yes       |Clerk|
| 2       | No        |Manager|
| 3       | No        |Analyst|
| 4       | Yes       |Clerk|

If NewEmp needs to be bitmap indexed, the content of bit map is shown as below

| NewEmp_values | Bitmap Indices    |
|---------|-----------|
| Yes       | 1001       |
| No       | 0110        |

Similarly for Job,

| Job_values | Bitmap Indices    |
|---------|-----------|
| Clerk       | 1001    |
| Manager       | 0100  |
| Analyst       | 0010  |

For ex, if we want to run this query

```sql
SELECT * 
        FROM Employee 
            WHERE NewEmp = "No" and Job = "Manager";
```

BitMap for "No"      --> 0110
BitMap for "Manager" --> 0100
                        ------
Result               --> 0100

#### Syntax for creating bitmap index

```sql
CREATE BITMAP INDEX index_New_Emp
        ON Employee (NewEmp);
```

Advantages -
- Efficiency in terms of insertion deletion and updation
- Faster retrieval of records

Disadvantages -
- Only suitable for large tables
- Bitmap Indexing is time consuming


### Composite index

```sql
# create index test_a_b on test(a,b);
```
left side of (a,b) matters.

------

```sql
# select c from test where a = 100;
```
for above query, `test_a_b` index is used.

------
```sql
# select c from test where b = 100;
```
for above query, `test_a_b` index is NOT used. Instead, parallel seq scan is performed.

------
```sql
# select c from test where a = 70 and b = 80;
```
for above query, `test_a_b` index is used.

------
```sql
# select c from test where a = 70 or b = 80;
```
for above query, `test_a_b` index is NOT AT ALL used. Instead, parallel seq scan is performed.


-------
Along with the above composite index, if we create below index on `b`
```sql
# create index test_b on test(b);
```

`# select c from test where a = 70 or b = 80;` query will use both `test_a_b` and `test_b`, does `BitmapOr` to fetch final results.


#### Extras

There is something called stats which is used when we do `explain analyze <query>`. So, sometimes when you do huge updates to the table and if stats are not updated properly, it might give incorrect results.

```sql
postgres=# analyze grades
```
The above command does statistics on the table.


### Create index concurrently 

Generally when an index is being created, you can not edit but you can comfortably read from the table. So, you can concurrently create an index in Postgres.


```sql
postgres=# create index concurrently g on grades(g);
```
This will take more cpu, memory and it can potentially fail.


### Bloom filters


Lets assume the situation where you want to check if an username exists in the DB. Naive way is to hit the DB and check all the time but this is too much. So, we can introduce some cache in between which would improve but still inefficient.

We can make use of hash function. After GET /username?q=jack, we hash the query `jack`, so we get `Hash(jack) % 64 = 63`. We will have a bitmap vector, where we set if there is an username whose Hash modulo 64. This will ofc have lot of collissions, but if there is some bit which is not set, it will improve the performance hugely. But, still when we have a lot of data, this is still inefficient. Cassandra uses this in implementation of consistent hashing. They use it internally.


### The cost of long running transactions

The cost of a long-running update transaction that eventually failed in Postgres (or any other database for that matter.

In Postgres, any DML transaction touching a row creates a new version of that row. if the row is referenced in indexes, those need to be updated with the new tuple id as well. There are exceptions with optimization such as heap only tuples (HOT) where the index doesn’t need to be updated but that only happens if the page where the row lives have enough space (fill factor < 100%)

If a long transaction that has updated millions of rows rolls back, then the new row versions created by this transaction (millions in my case) are now invalid and should NOT be read by any new transaction. You have many ways to address this, do you clean all dead rows eagerly on transaction rollback? Or do you do it lazily as a post-process? Or do you lock the table and clean those up until the database fully restarts?

Postgres does the lazy approach, a command called vacuum which is called periodically. Postgres attempts to remove dead rows and free up space on the page.

What's the harm of leaving those dead rows in? It's not really correctness issues at all, in fact, transactions know not to read those dead rows by checking the state of the transaction that created them. This is however an expensive check, the check to see if the transaction that created this row is committed or rolled back. Also, the fact that those dead rows live in disk pages with alive rows makes an IO not efficient as the database has to filter out dead rows. For example, a page may have contained 1000 rows, but only 1 live row and 999 dead rows, the database will make that IO but only will get a single row of it. Repeat that and you end up making more IOs. More IOs = slower performance.

Other databases do the eager approach and won’t let you even start the database before rolling back is successfully complete, using undo logs. Which one is right and which one is wrong? Here is the fun part! Nothing is wrong or right, it's all decisions that we engineers make. It's all fundamentals. It's up to you to understand and pick. Anything can work. You can make anything work if you know what you are dealing with.


### Clustered Index design

{READ LATER}

https://www.udemy.com/course/database-engines-crash-course/learn/lecture/30175448#learning-tools



