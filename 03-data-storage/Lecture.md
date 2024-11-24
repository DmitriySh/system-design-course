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
   - complexity: O(log n);
   - consist of: root node, internal nodes and leaves;
   - the root node has at least 2 children unless it is a leaf;
   - the data in the index is sorted in ascending order;
   - every internal node has at least `(m/2)` children, at most `(m)` children;
   - all leaves are at the same depth from the root;
   - number of keys in an internal node `(m/2) − 1`;
   - effective for operations: `<`,   `<=`,   `=`,   `>=`,   `>`,   `like 'abc%'`

B-tree history: were invented in Boeing Research Labs, for the purpose of efficiently managing index pages 
for large random-access files because only small chunks of the tree could fit in main memory; \
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
    - bit is a basic unit of information (0/1 or No/Yes), 
    - fits into memory (mostly);
    - useful for complex select queries involving multiple conditions that can benefit from combining to bitmap;
    - bitmap length = number of rows in the table;
    - separate bitmap for each column;
    - useful for columns with a low cardinality: if value is present in the row, set `1` to the desired bit or `0` otherwise;
    - query execution is combined using the bitwise operators AND, OR and NOT;
    - effective: 

Advantages: can speed up query performance, especially when dealing with large datasets and complex queries that involve multiple filters. \
Disadvantages: can slow down write performance on insert, update, or delete operations; \
Bitmap index note: PostgreSQL doesn't support bitmap indexes directly as a separate index type. \
Instead, it uses bitmap index scans internally when executing certain types of queries, particularly those that involve multiple conditions. 
```
Dating site: select a man, non-smoker, non-drinker, has a dog, single

[user_id][ man ][smoking][drinking][has_dog][single]
   126   | true | true   |  true   | false  | true |
   127   | true | false  |  false  | true   | true |
   128   | false| false  |  true   | false  | true |

CREATE INDEX idx_example ON table_name USING bitmap (column_name);   
bitmap index man:      110
bitmap index smoking:  100
bitmap index drinking: 101
bitmap index has_dog:  010
bitmap index single:   111


SELECT * 
FROM user 
WHERE man = true and smoking = false and drinking = false and has_dog = true and single = true
;
-- search row: 10011

vertical indexes
man:      110
   and
smoking:  100
   and
drinking: 101
   and
has_dog:  010
   and
single:   111
=============
2nd row has to be retrieved as a result:   010 => user_id=127
```



- `spatial grid index` = 


- `reversed index` =


- `functional index` = 


- `sparse index` = 


- `include index` =


- `cluster index` = 




## Transactions

## Message brokers

## Patterns of data storage and delivery data
