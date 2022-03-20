## B-Trees vs B+Trees

#### Full table scans

This is the slowest form of scan. 

#### Original B-Tree

B-Tree is a Balanced Tree.

https://use-the-index-luke.com/sql/anatomy/the-tree


**Tuples**: Rows/Tuples are synonyms.


- Each element has a key and a value.
- The value is usually data pointer to the row.
- Data pointer can point to primary key or tuple.
- Root Node, internal node and leaf nodes.
- A node = disk page.

#### How B-Trees improve performance

https://www.postgresql.eu/events/fosdem2020/sessions/session/2863/slides/278/PostgreSQL%20Conf%20FODEM%202020%20-%20Deep%20Dive%20to%20PostgreSQL%20Indexes.pdf


#### B+Tree

- Exactly like B-Tree but only stores keys in internal nodes.
- Values are only stored in leaf nodes.
- Internal nodes are smaller since they only store keys and they can fit more elements.
- Leaf nodes are "linked" so once you find a key you can find all values before and after that key.
- Great for range queries.

#### B+Tree DBMS Considerations

- Cost of leaf pointer (cheap)
- 1 Node fits a DBMS page (most DBMS)
- Can fit internal nodes easily in memory for fast traversal.
- Leaf nodes can live in data files in the heap.
- Most DBMS systems use B+Tree.

#### Storage Cost in Postgres vs MySQL

- B+Trees secondary index values can either point directly to the tuple (Postgres) or to the primary key (MySQL)
- If the primary key data type is expensive this can cause bloat in all secondary indexes for databases such as MySQL (InnoDB)
- Leaf nodes in MySQL (InnoDB) contains the full row since its an IOT/clustered index.x`