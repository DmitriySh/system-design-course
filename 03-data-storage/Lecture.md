03 - Data storage
=======

`Database` is a form of information organization where information is stored in a form convenient for operations: \
adding, updating, searching and deleting this information.

## Database types
- `Relational` = based on the relational model and data is structured in tables that contain rows and columns; \
  examples: PostgreSQL, MySQL; \
  features: relations between tables; indexes; transactions; ACID; OLTP
```
Query:
SELECT id, status
FROM user_info
WHERE id IN (20, 30);

Result:
 id | status 
----+--------
 20 | FIRST
 30 | SECOND
```

- `Column` = data is stored in columns rather than rows; \
  examples: ClickHouse, Vertica; \
  features: stores data in separate columns; faster search and extraction; horizontal scaling; data compression; OLAP
```
Query:
SELECT id, status
FROM user_info
WHERE id IN (20, 30);

Result:
id	status
20	FIRST
30	SECOND
```

- `Document-oriented` = designed for storing, retrieving and managing document-oriented information; \
  examples: MongoDB, CouchDB; \
  features: schemaless; all data in one document; horizontal scaling
```
Query:
db.user_info.find( 
  {id: {$in: [20, 30]} }, 
  {id: 1, status: 1} 
);

Result:
[
  {
    "id": 20,
    "status": "FIRST"
  },
  {
    "id": 30,
    "status": "SECOND"
  }
]
```

- `Fulltext search engine` = designed for full-text searching documents (json objects) by their content; \
  examples: Elasticsearch, Sphinx; \
  features: horizontal scaling; powerful full-text index searching; fast reading and low recording
```
Query:
{
  "query": {
    "terms": {
      "id": [10, 20]
    }
  },
  "fields": ["id", "status"]
}

Result:
{
  "hits": {
    "total": 2,
    "hits": [
      {
        "_source": {
          "id": 20,
          "status": "FIRST"
        }
      },
      {
        "_source": {
          "id": 30,
          "status": "SECOND"
        }
      }
    ]
  }
}
```

- `Key-Value` = designed to store, retrieving and managing associative arrays (key-value pairs) without complex data hierarchies and relationships; \
  examples: Redis, Memcached, Tarantool, RocksDB; \
  features: simple hash index; horizontal scaling; high performance; to cache data in memory
```
Query:
HMGET user_info:20 user_info:30 id status

Result:
[
  {
    "id": 20,
    "status": "EMPTY"
  },
  {
    "id": 30,
    "status": "ACTIVE"
  }
]
```

- distributed Key-Value storage of configuration and other service information; \
  examples: ZooKeeper, etcd


- `Wide-Column` = designed to store key-value data where the value consists of multiple columns and helps store related information; \
  examples: Cassandra, ScyllaDB; \
  features: schemaless; big data storage among different data centers; high (horizontal) scalability and reliability; high performance; data compression
```
Query:
SELECT id, status 
FROM user_info 
WHERE id IN (20, 30);

Result:
[
  {
    "id": 20,
    "status": "FIRST"
  },
  {
    "id": 30,
    "status": "SECOND"
  }
]
```

- `Graph` = designed to store data in graph structures with nodes, edges, and their properties.
  The relationships allow data in the store to be linked together directly and, in many cases, retrieved with one operation \
  examples: Neo4j, Tigergraph, Stardog; \
  use cases: social links between users, modeling and analysis of transport traffic, define recommendations to order the goods on marketplaces; \
  features: fast search in relations; relations are materialized and consuming a RAM opposed to RDBMS where relations building at runtime and consuming CPU
```
Query:
MATCH (u:UserInfo)
WHERE u.id IN (20, 30)
RETURN u.id, u.status;

Result:
[
  {
    "id": 20,
    "status": "FIRST"
  },
  {
    "id": 30,
    "status": "SECOND"
  }
]
```

- `Time series` = designed to collect data sequentially at regular intervals; time series consist of: timestamp and linked data \
  examples: InfluxDB, Prometheus, VictoriaMetrics; \
  use cases: currency quotes, CPU/RAM load, transport movement telemetry, server request statistics; \
  features: use compression and optimized storage formats in contrast to RDBMS; archive or delete old data
```
t(С)
 ^
 |            .
 |   ...     .
 | .    .   .
 |.      ...
 ----------------> time

    timestamp     | temrepature 
------------------+-------------
 10.01.2022 17:00 |    37.2
 10.01.2022 17:01 |    38.1
 10.01.2022 17:02 |    40.0
```

- `Blobe Store` = platform, provides a unified approach to object/block/file storage (binary large object) \
  examples: Ceph, AWS S3; \
  features: vertical and horizontal scaling; self-healing and self-managing storage nodes
```
# ceph: save the file to the created pool
$ rados -p poolName put cephFileName ./file.txt

# s3: save the file to the bucket
$ aws s3 cp ./file.txt s3://bucketName/file.txt
```

How to choose DB?
- transactions (if you need ACID for working with data);
- format defines the storage type;
- your own skills to work with;
- data access methods;
- community and technology maturity;
- frequency to change a data format;

You are making your own opinion about the type of database, and you need to test your proposal. \
Examples:
- need to store user's money --> **(relational database)**
- need a counter to track the number of video views --> **(key-value database)**
- need to store user profiles --> **(document-oriented database)**
- relationships between users --> **(graph database)**
- store source code files, images --> **(blobe storage)**
- need to store temperature meter readings --> **(time series database)**
- need store and seek user's posts and use full-text search, and caching most popular posts --> **(search engine, column, key-value and relational databases)**



## Database characteristics
[OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) (Online Analytical Processing) = organizing the
database characterized by long-time transactions and much more complex queries to huge amounts of data for the purpose of
business intelligence or reporting. OLAP workload creates by batching updates and table scans; \
examples: ClickHouse, DuckDB, Druid

[OLTP](https://en.wikipedia.org/wiki/Online_transaction_processing) (Online Transaction Processing) = organizing the
database works with a large flow of small-time transactions and responds immediately to user requests. 
OLTP workload creates by CRUD operations; \
examples: PostgreSQL, MySQL, MongoDB, Redis

`OLAP` database stores duplicate data from `OLTP` for analytics
```
  analytic requests
              |
              |
              ⌵
[OLTP] ---> [OLAP]
  ^
  |
  |
client requests
```

[HTAP](https://en.wikipedia.org/wiki/Hybrid_transactional/analytical_processing) (hybrid transaction/analytics processing) = designed for
integration both analytics and transactions without duplicating data into a separate database, instant applying modified data and use it in reports; \
examples: Greenplum, Tarantool Column Store; \
features: `HTAP` uses memory and column type of storage combined with row-based type; \
use cases: the buyer could immediately receive new relevant offers based on past purchases
```
analytic requests
      |
      |
      ⌵
    [HTAP]
      ^
      |
      |
client requests
```

`In-memory database` = is a database management system that primarily relies on RAM/NVRAM; \
examples: Redis, Memcached, Hazelcast, Ignite; \
features: faster than disk-optimized databases; internal optimization algorithms are simpler and execute fewer CPU instructions; \
use cases: if response time is critical (telecommunications network equipment, stock exchange or payment systems)

`Persistent database` = is a database management system that primarily relies on storage media that are designed 
to retain data even when the power is turned off (HDD, SSD, Flash memory); \
examples: PostgreSQL, MongoDB, ClickHouse; \
features: handle large amounts of data; can be scaled up or down; more complex to design and implement

`Embedded database` = is a database management system that completely embedded within a host application process; \
examples: H2, SQLite, DuckDB, Derby; \
features: portable and not require a separate server; can be integrated into IoT devices

`Single-file database` = is a database management system that stores the structure and all data in one file on the file system; \
examples: H2, SQLite, DuckDB; \
features: simplifies management, distribution and deployment of database

`Append-only database` =  is a database management system that storage can only append new data and can't modify/delete old; \
examples: LSM-Tree for indexes; \
features: implement immutable objects, maximizing concurrency, all writes go to the end of the file



## Indexes
`Index` = is an object into DBMS (data structure). \
 - advantages: 
   - main goal is an improves the speed of data retrieval operations
 - disadvantages: 
   - each insert/update/delete takes time to normalize the index data structure;
   - use additional memory;
   - complicates the query execution plan;

`Index Selectivity` = is the percentage of rows in a table that have the same value for the indexed key. \
If selectivity is **low/poor** (get _a lot of records_), database will prefer to perform a **full scan**. \
If selectivity is **high** (get _several records or one_), database will prefer to use the **index scan**.

```
(selected_rows / total_rows) * 100 = percent_selectivity

example 1:
selected_rows = 5
total_rows = 31 591
percent_selectivity = (5 / 31591) * 100 = 0,016%

example 2:
selected_rows = 15 350
total_rows = 31 591
percent_selectivity = (15350 / 31591) * 100 = 48,59%
```

Data structure:
 - `b-tree` (balanced tree) = is a tree which satisfies the following properties
   - complexity: O(log(n));
   - effective for operations: `<`,   `<=`,   `=`,   `>=`,   `>`,   `like 'abc%'`
   - consist of: root node, internal nodes and leaves;
   - the root node has at least 2 children unless it is a leaf;
   - the data in the index is sorted in ascending order;
   - every internal node has at least `(m/2)` children, at most `(m)` children;
   - all leaves are at the same depth from the root;
   - number of keys in an internal node `(m/2) − 1`;

B-tree history: were invented in Boeing Research Labs, for the purpose of efficiently managing index pages 
for large random-access files because the whole tree cannot be kept in main memory and only small chunks; \
B-tree features: disk reads by blocks in 4Kb/8Kb, and this could be a page with nodes and key values; 
data structure specially designed to work efficiently with disk memory and minimize input-output operations. \
B-tree has fewer node transitions than a binary tree which reduces the number of disk access needed to search a key.
```
            --------------[500]--------------
            |                               |
            ⌵                               ⌵
   -----[20, 150]------           -------[20, 150]-------
   |        |         |           |          |          |
   ⌵        ⌵         ⌵           ⌵          ⌵          ⌵
[5, 15] [25, 100] [175, 450]  [560, 590] [650, 775] [905, 1020]
```

 - `hash index` (hash table) = is an associative array (dictionary or map) that maps keys to values:
   - complexity: O(1);
   - effective for operations: `=`;
   - uses a hash function to compute an index by key into an array of buckets and save/get the value into/from;
   - key value is not stored in index;
   - hash collisions is possible, where the hash function generates the same index for more than one key (bucket 1 divides to bucket 1.1, bucket 1.2);
   - stores a 32-bit hash code;
   - array consumes memory and grows in size up to x2;

Hash index history: (january 1953) Hans Peter Luhn wrote an internal IBM memorandum that used hashing with chaining;
open addressing was proposed by A. D. Linh, building on Luhn's memorandum
Hash index features: is stored on disk; all the necessary information is laid out on the pages (metapage, bucketpage, overflow page, bitmap page)
```
hash(key) => index

index =     0    1   2     3    ... N
value =  [drink][  ][  ][orange]...[  ]
```

 - `bitmap index` = is a technique for indexing scalar data
     - complexity: O(n);
     - effective for operations: `=`;
     - bit is a basic unit of information (0/1 or No/Yes), 
     - fits into memory (mostly);
     - useful for complex select queries involving multiple conditions that can benefit from combining to bitmap;
     - bitmap length = number of rows in the table;
     - separate bitmap for each column;
     - useful for columns with a low cardinality: if value is present in the row, set `1` to the desired bit or `0` otherwise;
     - query execution is combined using the bitwise operators AND, OR and NOT;

Advantages: can quickly determine which rows satisfy the query by performing a bitwise operation 
on the corresponding bitmaps, especially when dealing with large datasets and complex queries that involve multiple filters. \
Disadvantages: can slow down write performance on insert, update, or delete operations; \
Bitmap index note: PostgreSQL doesn't support bitmap indexes directly as a separate index type. \
Instead, it uses bitmap index scans internally when executing certain types of queries, particularly those that involve multiple conditions. 
```
Dating site: select a man, non-smoker, non-drinker, has a medium dog, single

[user_id][ gender ][smoking][drinking][  dog  ][single]
   101   | man    | true    |  true   | small  | true |
   102   | man    | false   |  false  | medium | true |
   103   | woman  | false   |  true   | big    | true |
   104   | woman  | false   |  true   | null   | false|

CREATE INDEX idx_example 
ON table_name 
USING bitmap (column_name);  

gender value      Bitmap Index
-------------     ------------
man            =  1100
woman          =  0011

smoking value     Bitmap Index
-------------     ------------
true            =  1000
false           =  0111

drinking value    Bitmap Index
-------------     ------------
true            =  1011
false           =  0100

dog value         Bitmap Index
-------------     ------------
small           =  1000
medium          =  0100
big             =  0010
null            =  0001

single value      Bitmap Index
-------------     ------------
true            =  1110
false           =  0001

SELECT * 
FROM user 
WHERE gender = 'man' and smoking = false and drinking = false and dog = 'medium' and single = true
;

gender = 'man'       =>  1100
                         and
smoking = false      =>  0111
                         and
drinking = false     =>  0100
                         and
dog = 'medium'       =>  0100
                         and
single = true        =>  1110
=============        ========
result               =>  0100

Row with user_id=102 has to be retrieved as a result

[user_id][ gender ][smoking][drinking][  dog  ][single]
   102   | man    | false   |  false  | medium | true |
```

 - `spatial grid index` = is a structure for organizing and searching for spatial objects in the appropriate grid cells:
   - complexity: O(n*log(n));
   - it can quickly identify close values in 2 or more dimensions;
   - performs indexing the bounding boxes of the features instead of classic hierarchical tree index \
     based on the values of the column being indexed;
   - implementation: R-Tree (regions tree) index, Q-Tree (quadtree) index.

Spatial grid index can help find an answer of question: - "Find all museums within 2 km of my current location". \
`Point quadtree`: a portion of a two-dimensional space where each internal node is a square that is divided into exactly 4 equal children (nodes) \
used to store information about points on a plane; `Key` expresses coordinates (x,y), `value` is a custom associated value (user id, location name, other)
```
                         1
 |-----------|-----------|
 |  .        |      .    |
 |       .   |     .     |
 |   .       |0,5      . |
 |-----|-----|-----------|
 | .   |0,25 | .         |
 |--|--|-----|   .   .   |
 |--|--|   . |   .       |
 |--|--|-----|-----------|
 0
 
 [user_id][  x  ][  y  ]
   102    | 0.1  | 0.7
   132    | 0.2  | 0.2
   342    | 0.4  | 0.1
```

`R-Tree`: a balanced search tree data structures (like B-tree) for indexing multidimensional information such as geographical coordinates; \
organizes the data in pages, and is designed for storage on disk; break up data into intersecting rectangles, and sub-rectangles, and sub-sub rectangles
```
     A
     |-----------|
     |  d       e|         C
B    |       |---|---------|
|----|--|    |   |        i|
|    |  |  f |   |         |
|g   |--|----|---|   j     |
|       |    |-------------|
|      h|
|-------|

     --------[A][B][C][ ]
     |           |  |-----------
     |           |             |
     ⌵           ⌵             ⌵
[d][e][f][ ]  [g][h][ ][ ]  [i][j][ ][ ]
```


 - `reverse index` = is a B-Tree index literally reverse the bytes of the key value in the index to reduce block contention \
   on sequence generated keys:
     - complexity: O(log(n));
     - values from an auto-incrementing index will end up in different blocks of a b-tree index;
     - implementation: via functional index.

Advantages: 
 - important in high volume transaction processing systems;
 - speeding up insertions;
Disadvantage: can't perform range scans because the index entries are scattered all over instead of being stored sequentially (in a classic B-Tree).
```
source   | reverse
---------|---------
0000 (0) | 0000 (0)
0001 (1) | 1000 (8)
0010 (2) | 0100 (4)
0011 (3) | 1100 (12)
0100 (4) | 0010 (2)
0101 (5) | 1010 (10)
0110 (6) | 0110 (6)
0111 (7) | 1110 (14)
1000 (8) | 0001 (1)
1001 (9) | 1001 (9)
1010 (10)| 0101 (5)


source: 0,1,2,3,4,5,6,7,8,9,10
       ----------[3]----------
       |                     |
       ⌵                     ⌵
   ---[1]---        -------[5, 7]-------
   |        |       |         |        |
   ⌵        ⌵       ⌵         ⌵        ⌵
  [0]      [2]     [4]       [6]    [8, 9, 10]


reverse: 0,8,4,12,2,10,6,14,1,9,5
    ---------[4,8,10]---------
    |        |       |       |
    ⌵        ⌵       ⌵       ⌵
 [0,1,2]   [5, 6]   [9]   [12,14]
```


 - `inverted index` = is an index data structure mapping 'words' to its locations in a 'document' or a set of 'documents'; \
   it is called an inverted index because it is simply an inversion of the forward index.
     - complexity: depends on index data structure;
     - implementation: it could be a form of a hash table or distributed hash table for large indexes;
     - widely used in search engines, database systems where efficient text search is required; \
       useful for large collections of documents, where searching through all the documents would be prohibitively slow

Advantages:
 - fast full-text searches;
 - can be compressed to reduce storage requirements. \
Disadvantages:
 - large storage overhead and high maintenance costs on updating/deleting/inserting;
 - the records are retrieved in the order in which they occur in the inverted lists instead of order of expected usefulness.

The `forward index` stores a list of words for each document:
```
  id  | document
----- |----------
  0   |it is what it is
  1   |what it is
  2   |it is a banana  
```

The `inverted index: record-level` stores a list of the documents containing each word:
```
word  | document_ids
----- |-------------
  a   | [2]
banana| [2]
  is  | [0,1,2]
  it  | [0,1,2]
 what | [0,1]
```

The `inverted index: word-level` stores a list of the documents containing each word and their positions within documents:
```
word  | document_ids
----- |-------------
  a   | [(2,2)]
banana| [(2,3)]
  is  | [(0,1), (1,2), (2,1)]
  it  | [(0,0), (0,3), (1,1), (2,0)]
 what | [(0,2), (1,0)]
```

The index may take a different approach and create tokens from words.
Example: "johnie" with 3 letter n-grams are:
- joh,
- ohn,
- hni,
- nie
```sql
CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT,
    column tsvector
);

CREATE INDEX documents_column_idx 
ON documents 
USING GIN (column);
```


 - `functional index` = is an index that is based on the result of a function applied to one or more columns in a table; \
   allows to index values that are not stored directly in the table itself.
     - complexity: depends on index data structure;
     - effective for operations: depends on index data structure.

Example: you have a 'scorecard' of students by mathematics, physics, language, and you need find the top 10 students with the best average score. \
Advantages: avoid performing unnecessary calculations in complex queries and full scans. \
Disadvantages: index needs to be updated every time the table is modified (insert and update).
```sql
CREATE INDEX functional_idx
ON table_name( ((physics + mathematics + language)/3) );

EXPLAIN ANALYZE
SELECT id, name, ((physics + mathematics + language)/3) as average_score
FROM table_name
ORDER BY average_score DESC
LIMIT 10;
         QUERY PLAN                     
---------------------------
 Limit  (cost=0.42..1.00 rows=10 width=22) (actual time=0.060..0.072 rows=10 loops=1)
   ->  Index Scan Backward using functional_idx on table_name  (cost=0.42..57448.53 rows=1000005 width=22) (actual time=0.058..0.068 rows=10 loops=1)
 Planning Time: 0.344 ms
 Execution Time: 0.086 ms
```


 - `sparse index` = is an index that contains an index entry only for some records and not for all rows
     - complexity: depends on index data structure;
     - effective for operations: depends on index data structure;
     - two different approaches to organizing and accessing data: **dense index** and **sparse index** \
       depends on data operations;
     - PostgreSQL doesn't support **sparse index** and use **partial index** instead.

The `dense index` contains an index record for every search key value:
 - quick access to records;
 - effective for range searches since each key value has an entry;
 - require a significant amount of storage space;
 - slows down performance during insert/update operations because the index will be updated more frequently.

The `sparse index` contains an index record for not every search key value:
 - searching access may involve sequence scan if index key misses;
 - might not be as effective for range queries;
 - need less storage space;
 - less overhead for insert/update operations because the index cover not all records;
 - reduces the number of I/O operations for certain types of requests;
 - records need to be clustered for efficient searching;
```
  -----------
  |         |
--|---------|-- index key = 100
  |         |
--|---------|-- index key = 200
  |         |
--|---------|-- index key = 300
  |         |
  -----------
```


 - `covering index` = is an index specifically designed to include necessary columns so that filtering use data \
   within the index and without visiting the heap;
     - complexity: depends on index data structure;
     - effective for operations: depends on index data structure;
     - include columns are just "payload" and are not part of the "search key";
     - it's merely stored in the index and is not interpreted by the index machinery;
     - speeding up queries with **index-only scans**;

Advantages:
 - queries can be executed faster;
 - requires less memory to process the data;
 - reduced loading on the disk subsystem;

Disadvantages:
 - large storage overhead;
 - high maintenance costs to support "include fields" in the index;
 - not all index types support covering indexes;
```sql
CREATE INDEX x_y_idx ON table_name(x) INCLUDE (y);

SELECT y 
FROM table_name 
WHERE x = 'key';
```


 - `cluster index` = is a database indexing technique that is used to physically arrange the data in a table \
   based on the values of the clustered index key (ordering);
     - complexity: depends on index data structure;
     - effective for operations: depends on index data structure;
     - table can have only one clustered index;
     - if data is frequently added to the table, clustering indexing may not be the best choice.

Advantages: database can more efficiently retrieve data by range (reducing disk I/O). \
Disadvantages: database must reorganize the data to reflect every change.



## Additional features
 - `Stored procedure` = is a group of SQL queries that can be saved and reused multiple times:
   - can be implemented in a variety of programming languages (PL/pgSQL, SQL/PSM, Java, C and others);
   - it is compiled, stored in the database and is close to the data, which is highly efficient;
   - typically includes flow control statements: IF, WHILE, LOOP, REPEAT and CASE;
   - can receive variables, declare variables and return result sets;
   - works in the same transaction that the sql statement;
```sql
CREATE OR REPLACE PROCEDURE select_expensive_products(price_threshold double precision)
LANGUAGE plpgsql
AS $$
DECLARE
    product_record products%ROWTYPE;
BEGIN
    -- Check precision
    IF price_threshold > 0 THEN
        FOR product_record IN 
            SELECT * FROM products WHERE price > price_threshold
        LOOP
            RAISE NOTICE 'Found product: %, price: %', product_record.name, product_record.price;
        END LOOP;
    ELSE
        RAISE NOTICE 'The price_threshold variable should not be negative or zero.';
    END IF;
END
$$;

-- Invoke procedure
CALL select_expensive_products(100.0);
```


 - `Trigger` = is a special type of stored procedure that automatically executes or fires when certain events occur in a database:
   - can be executed before/after any INSERT, UPDATE, or DELETE operation;
   - can be executed instead of INSERT, UPDATE, or DELETE operations;
   - have access to an old and new row set;
   - can be attached to tables, views, and foreign tables.
```sql
CREATE OR REPLACE FUNCTION before_update_salary()
RETURNS TRIGGER AS $$
BEGIN
 -- Check the growth of the new salary value
 IF NEW.salary > OLD.salary THEN
     -- Insert new row into salary_history table
     INSERT INTO salary_history (employee_id, OLD.salary, NEW.salary)
     VALUES (OLD.id, OLD.salary, NEW.salary);
 END IF;

 -- Return new row
 RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Register trigger (before update)
CREATE TRIGGER increase_salary
    BEFORE UPDATE ON employees
    FOR EACH ROW
    EXECUTE FUNCTION before_update_salary();
```


 - `View` = is not a physically materialized result table of a query:
   - alias for a query;
   - helps to hide the complexity of the original query;
   - data is not updatable.
   - examples: RDBMS, MongoDB
```sql
CREATE VIEW books_count AS
SELECT book_id, title, sum(quantity) 
FROM books as b, book_copies as bc
WHERE b.book_id = bc.book_id
GROUP BY b.book_id;

-- Invoke view
SELECT * FROM books_count;
```

 - `Materialized view` = is a physically materialized result table of a query:
   - remembers the query used to initialize the view, so that it can be refreshed later upon demand;
   - data can be auto-updatable.
   - examples: RDBMS, MongoDB, Cassandra
```sql
CREATE MATERIALIZED VIEW books_count AS
SELECT book_id, title, sum(quantity) 
FROM books as b, book_copies as bc
WHERE b.book_id = bc.book_id
GROUP BY b.book_id;

-- Invoke view
SELECT * FROM books_count;

-- Refresh the materialized view data
REFRESH MATERIALIZED VIEW books_count;
```


 - `Watch API` = is an event-based interface for asynchronously monitoring of data changes or special events happen:
   - examples: etcd3, Elasticsearch

## Transactions
ACID - properties describe the major guarantees of the transaction paradigm even in the event of errors, power failures, etc.
 - `atomicity` = guarantees that each transaction is treated as a single "unit", which either succeeds completely, or fails completely;
 - `consistency` = ensures that a transaction can only bring the database from one valid state to another, prevents database corruption after an illegal transaction; \
   constraints: 
   - not null value, 
   - unique value, 
   - primary/foreign key, 
   - check condition for value, 
   - default value;
 - `isolation` levels = ensure that transactions can execute concurrently and don't interfere with each other to read, write data:
   - if higher the transaction isolation level, than lower the database throughput;
   - isolation resolution methods:
     - `2 phase locking` (2pl) = ?
     - `multiversion concurrency control` (mvcc) = ?
```
                  lost update | dirty read | non-repeatable read | phantom reads | other anomalies
Read Uncommitted       -            +                +                   +                +
Read Committed         -            -                +                   +                +
Repeatable Read        -            -                -                   +                +
Serializable           -            -                -                   -                -
```
 - `durability` = guarantees that once a transaction has been committed, it will remain committed even in the case of a system failure.



## Message brokers


## Patterns of data storage and delivery data
