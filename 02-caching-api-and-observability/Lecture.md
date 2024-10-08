02 - Caching, API and Observability
=======

## Caching
This is the practice of temporarily storing frequently used or calculated data in memory or on a disk. \
The main goal is to speed up the response, not to keep the increased load.

Functionality:
 - reducing the response time of services;
 - reducing load on third-party services;
 - reuse previously obtained or calculated data;
 - stabilization of operation in case of short-term system failures.

Key terms:
 - `Cache miss` = the requested key was not found in the cache
 - `Cache Hit` = the requested key was found in the cache
 - `Hit ratio` = the percentage of hits of requests to the cache (effectiveness of caching)
 - `Hot key` = a key that is accessed frequently, or with a high frequency
 - `Cache warming` = proactive technique to preload data into a cache before it is actually needed by the system
 - `Cache invalidation` = push away the data from the cache memory when the data is outdated

What data should be cached?
 - can change frequently (lifetime in seconds)
 - can change not frequently (lifetime in minutes/hours)
 - can change rarely (lifetime in days/months)

Is caching <span style="color:green">useful</span> or <span style="color:red">harmful</span>?
 - <span style="color:green">useful</span>: caching helps to speed up the response to a request;
 - <span style="color:green">useful</span>: at startup with an empty cache the system may degrade. Warming up the cache can help;
   - selection of popular queries: goods, users;
   - frequently requested reference data that doesn't change;
 - <span style="color:red">harmful</span>: the data was updated in the database but not in the cache, and all requests continue to retrieve the old data before invalidation;
 - <span style="color:red">harmful</span>: frequent cache misses increase response time.
```
CacheMissRate   > 80%, value = (0,8..1,0]
DBAccessTime    = 100ms
CacheAccessTime = 20ms
---

AverageTime = DBAccessTime * CacheMissRate + CacheAccessTime = 100 * 0,81 + 20 = 101 ms
(AverageTime > DBAccessTime) -> caching is harmful
```

Types:
 - internal caching
   - high speed;
   - without external network requests;
   - no costs for data marshaling/unmarshalling;
   - no scaling or difficulties in horizontal scaling:
     - separate all keys by segments and store them in parts on servers; \
       a client needs to use a balancer or sticky sessions to avoid misses
     - duplicate keys/values on all servers.
   - need to warm up cache after service failure.
```
  -----------    -----------    -----------
  | server1 |    | server2 |    | server3 |
  |    +    |    |    +    |    |    +    |
  |  cache  |    |  cache  |    |  cache  |
  -----------    -----------    -----------
```
 - external caching
   - not high speed; 
   - access with external network requests;
   - ability to store large amounts of data;
   - ability to easily scale keys/values horizontally;
   - cache data won't be lost if the service goes down;
   - easy to warm up cache and simple invalidation logic;
```
   [cache1]          ---->[cache2]<---
       ^             |               |
       |             |               |
  -----------    -----------    -----------
  |         |    |         |    |         |
  | server1 |    | server2 |    | server3 |
  |         |    |         |    |         |
  -----------    -----------    -----------
```

Strategies:
 - `cache aside: read` = data puts into the cache only after the data is requested \
   advantages: cache contains only data that the service actually requests (helps keep the cache size cost-effective); \
   advantages: simple implementation and immediate performance gains; \
   disadvantage: the data is loaded into the cache only after a cache miss, additional round trips to the cache and database are needed.; \
   disadvantage: the database and cache may be inconsistent if a key was cached but the value was later updated in the database.
```
      -------------(5)-------------
      |                           |
      |                           ⌵
  -----------                 -----------
  |         | ------(1)-----> |         |
  | server  |                 |  cache  |
  |         | <-----(2)------ |         |
  -----------                 -----------
     |   ^
    (3)  |
     |  (4)
     ⌵   |
  -----------
  |         |
  |database |
  |         |
  -----------

(1) - read data by 'key' form the cache;
(2) - return data from the cache;

if cache miss:
(3) - read source data from database;
(4) - return source data from database;
(5) - save data into the cache;
```
 - `cache aside: write` = saved to the cache after insertion into the database
   advantages: database and cache remain consistent
   disadvantage: warming up is necessary if the caching service failed and later started empty 
```
  -----------                 -----------
  |         |                 |         |
  | server  | ------(3)-----> |  cache  |
  |         |                 |         |
  -----------                 -----------
     |   ^
    (1)  |
     |  (2)
     ⌵   |
  -----------
  |         |
  |database |
  |         |
  -----------

(1) - insert/update data into the database;
(2) - response from database about successful update;
(3) - save data into the cache;
```
 - `cache through: read` = all requests go through the cache and update it immediately; \
   wait when the primary database is updated (blocking mode) or don't wait when update (async mode). \
   advantages: reducing the load on the database \
   advantages: the cache is up-to-date, which prevents data inconsistency with the database \
   disadvantage: additional waiting time for a response
```
  -----------                 -----------                 -----------
  |         | ------(1)-----> |         | ------(3)-----> |         |
  | server  |                 |  cache  |                 |database |
  |         | <-----(2)------ |         | <-----(4)------ |         |
  -----------                 -----------                 -----------

(1) - read data by 'key' from the cache;
(2) - return data from the cache;

if cache miss:
(3) - read source data from database;
(4) - return source data from database and save data into the cache;
```
 - `cache through: write`
```
  -----------                 -----------                 -----------
  |         | ------(1)-----> |         | ------(2)-----> |         |
  | server  |                 |  cache  |                 |database |
  |         | <-----(4)------ |         | <-----(3)------ |         |
  -----------                 -----------                 -----------

(1) - save data into the cache;
(2) - insert/update data into the database;
(3) - response from database about successful update and save data into the cache;
(4) - return data from the cache;
```
 - `cache ahead` = requests always go only to the cache, never going directly to the DB


## API


## Observability

