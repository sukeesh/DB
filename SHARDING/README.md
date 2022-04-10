### What is Sharding?

If there are many rows, DB Indexes grows heavily. Queries become very slow. So, split 1 million rows table into 5 database instances based on a Shard/Partition key. So, whenever there is a query with key, consistent hashing is done to find out 

### Consistent Hashing

Given an input, Consistent hashing always gives back the same server address consistently. Consistent Hashing Ring (Check Gaurav Sen's video)


### Horizontal Partitioning vs Sharding

- HP Splits big table into multiple tables in the same database.
- Sharding splits big table into multiple tables across multiple database servers.
- HP table name changes (or schema).
- Sharding everything is the same but server changes.


### Writing and Querying a Shard

{DO A PROJECT in Golang} 


### Pros of Sharding

- Scalability
	- Data
	- Memory
- Security (users can access certain shards)
- Optimal and smaller index size


### Cons of Sharding

- Complex client (aware of the shard)
- Transactions across shards problem
	- Eg: Atomically insert into different shards. Thats impossible.
- Rollbacks
- Schema changes are hard
- Joins
- Has to be something you know in the query


### When should you consider sharding a database?


