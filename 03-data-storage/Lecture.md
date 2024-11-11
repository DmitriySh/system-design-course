03 - Data storage
=======

`Database` is a form of information organization where information is stored in a form convenient for operations: \
adding, updating, searching and deleting this information.

## Database types
 - `Relational` = based on the relational model and data is structured in tables that contain rows and columns; \
   examples: PostgreSQL, MySQL; \
   features: relations between tables, indexes, ACID
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
   features: stores data in separate columns; faster search and extraction, horizontal scaling, data compression; OLAP \
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
   features: schemaless, all data in one document, horizontal scaling
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
   features: horizontal scaling, powerful full-text index searching; fast reading and low recording
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
   features: simple hash index; horizontal scaling, high performance, to cache data in memory
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
   features: schemaless; big data storage among different data centers; high (horizontal) scalability and reliability; high performance; data compression;
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
   features: fast search in relations; relations are materialized and consuming a RAM opposed to RDBMS where relations building at runtime and consuming CPU; \
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

- `Time series` = designed to collect data sequentially at regular intervals; \
  examples: InfluxDB, Prometheus, VictoriaMetrics; \
  use cases: currency quotes, CPU/RAM load, transport movement telemetry, server request statistics; \

## Database characteristics


## Indexes


## Transactions


## Message brokers


## Patterns of data storage and delivery data
