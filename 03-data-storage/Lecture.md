03 - Data storage
=======

`Database` is a form of information organization where information is stored in a form convenient for operations: adding, updating, searching and deleting this information

## Database types
 - `Relational` = based on the relational model and data is structured in tables that contain rows and columns. \
   examples: PostgreSQL, MySQL; \
   features: relations between tables, indexes, ACID
```sql
SELECT id, status
FROM user_info
WHERE id IN (20, 30);
```
 - `Column` = data is stored in columns rather than rows (NoSQL DB) \
   examples: ClickHouse, Vertica; \
   features: high performance, horizontal scaling, data compression; \
```
SELECT id, status
FROM user_info
WHERE id IN (20, 30);
```
 - `Document-oriented` = designed for storing, retrieving and managing document-oriented information (NoSQL DB)
   examples: MongoDB, CouchDB; \
   features: schemaless, all data in one document, indexes, horizontal scaling
```
db.user_info.find( 
  {id: {$in: [20, 30]} }, 
  {id: 1, status: 1} 
);
```

 - `Fulltext search engine` = designed for full-text searching documents (json objects) by their content; \
   examples: Elasticsearch, Sphinx; \
   features: horizontal scaling, high performance, powerful full-text index searching
```
{
  "query": {
    "terms": {
      "id": [10, 20]
    }
  },
  "fields": ["id", "status"]
}
```

 - `Key-Value` = designed for storing, retrieving and managing associative arrays (key-value pairs) without complex data hierarchies and relationships
?

## Database characteristics


## Indexes


## Transactions


## Message brokers


## Patterns of data storage and delivery data
