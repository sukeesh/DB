### How tables and indexes are stored on disk?


#### Page

Depending on storage model, the rows are stored and read in logical pages. The DB doesn't read a single row, it reads a page or more in a single IO and we get a lot of rows in that IO. Each page has a size (e.g. 8KB in postgres, 16KB in MySQL). Assume each page holds 3 rows in this example, with 1001 rows you will have 1001/3 = 333~ pages


#### Heap

Heap is data structure where the table is stored with all its pages one after another. Collection of pages.


#### Index

An index is another data structure separate from the heap that has "pointers" to the heap. it has part of the data and used to quickly search for something. You can index on one column or more. Once you find a value of the index, you go to the heap to fetch more information where everything is there. Popular data structore for index is b-trees.


#### Row-Based vs Column-Based Databases

### Row-Oriented Database

- Tables are stored as rows in disk. A single block io read to the table fetches multiple rows with all their columns. More IOs are required to find a particular row in a table scan but once you find the row you get all columns for that row.

### Column-Oriented Database

- Tables are stored as columns first in disk. A single block io read to the table fetches multiple columns with all matching rows. Less IOs are required to get more values of a given column. But working with multiple columns require more IOs.

### Pros & Cons

| Row-Based                         | Column-Based                        |
|-----------------------------------|-------------------------------------|
| Optimal for read/writes           | Writes are slower                   |
| OLTP                              | OLAP                                |
| Compression isn't efficient       | Compress greatly                    |
| Aggregation isn't efficient       | Amazing for aggregation             |
| Efficient queries w/multi-columns | Inefficient queries w/multi-columns |
|  | Perfect for online analytics stuff, where you don't write stuff |

