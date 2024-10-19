02 - Caching, API and Observability
=======

## Caching
This is the practice of temporarily storing frequently used or calculated data in memory or on a disk. \
The main goal is to **speed up the response (RPS)**, not to keep the increased load. The most well-known problem of working 
with a cache is **data invalidation**. \
If your program can work without caching, don't use it.

Cache - is a key-value storage in memory with higher speed data access than the main database.

Functionality:
 - reducing the response time of services;
 - reducing load on third-party services;
 - reuse previously obtained or calculated data;
 - stabilization of operation in case of short-term system failures.

Key terms:
 - `Cache miss` = the requested key was not found in the cache
 - `Cache hit` = the requested key was found in the cache
 - `Hit ratio` = percentage of hits of requests to the cache (caching is more effective if value closer to 100%)
 - `Hot key` = a key that is accessed frequently, or with a high frequency
 - `Cache warming` = proactive technique to preload data into a cache before it is actually needed by the system
 - `Cache invalidation` = push away the data from the cache memory when the data is outdated
 - `Expired rate` = percentage of records deleted after TTL expires (caching is ineffective if value higher 50%)
 - `Eviction rate` = percentage of cache entries evicted when the memory limit is reached (caching is ineffective if value higher 50%)

What data should be cached?
 - can change frequently (lifetime in seconds) - no need to be cached
 - can change not frequently (lifetime in minutes/hours) - caching could help or not
 - can change rarely (lifetime in days/months) - caching should help

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

 - `cache aside: write` = saved to the cache after insertion into the database \
   advantages: database and cache remain consistent \
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
(2) - response from database about successful insert/update;
(3) - save data into the cache;
```

 - `cache through: read` = all read requests go through the cache and update it immediately; \
   advantages: reducing the load on the database \
   advantages: the cache is up-to-date, which prevents data inconsistency with the database \
   disadvantage: additional waiting time for a response if cache miss data
```
  -----------                 -----------                 -----------
  |         | ======(1)=====> |         | ------(3)-----> |         |
  | server  |                 |  cache  |                 |database |
  |         | <=====(2)====== |         | <-----(4)------ |         |
  -----------                 -----------                 -----------

(1) - read data by 'key' from the cache;
(2) - return data from the cache;

if cache miss:
(3) - read source data from database;
(4) - return source data from database and save data into the cache.
```

 - `cache through: write` = all write requests go through the cache and then update data in database; \
   wait when the primary database is updated (blocking mode) or don't wait when update (async mode). \
   advantages: helps cache maintain consistency with the primary database \
   disadvantage: additional waiting time to write data with blocking mode
```
  -----------                 -----------                 -----------
  |         | ======(1)=====> |         | ======(2)=====> |         |
  | server  |                 |  cache  |                 |database |
  |         | <-----(4)------ |         | <-----(3)------ |         |
  -----------                 -----------                 -----------

(1) - write data by 'key' into the cache;
(2) - insert/update data into the database;
(3) - response from database about successful update and commit data into the cache;
(4) - response from cache about successful write.
```

 - `cache ahead` = read requests always go only to the cache, never use the database data directly
```
  -----------                 -----------                 -----------
  |         | ======(1)=====> |         |                 |         |
  | server  |                 |  cache  | ------(3)-----> |database |
  |         | <=====(2)====== |         |                 |         |
  -----------                 -----------                 -----------

(1) - read data by 'key' from the cache;
(2) - return data from the cache;
(3) - periodically upload data from the database.
```

The choice of **caching strategy** depends on the nature of the load (read or write) and necessity to update the data.


Cache replacement algorithms:
 - `RR` (random replacement) = replace randomly selected object when the maximum capacity is reached; \
   doesn't keep history of objects; \
   very easy to implement;
 - `FIFO` = objects are replaced in the order they are added to the cache since the older if the maximum capacity is reached (one-way queue); \
   popular elements will be evicted when they're old enough; \
   simple and low-cost to implement;
```
1add   3add   0add   5add   6add   0read  3add
| |    | |    |0|    |0|    |0|    [0]    [3]
| | -> |3| -> |3| -> |3| -> [6] -> |6| -> |6|
|1|    |1|    |1|    [5]    |5|    |5|    |5|
```
 - `LIFO` = objects are replaced in the reverse order they are added to the cache if the maximum capacity is reached (stack); \
   popular elements can be evicted if they are young enough; \
   simple and low-cost to implement;
```
1add   3add   0add   5add   6add   1read  3add
| |    | |    |0|    [5]    |5|    |5|    |5|
| | -> |3| -> |3| -> |3| -> [6] -> |6| -> |6|
|1|    |1|    |1|    |1|    |1|    [1]    [3]
```
 - `LRU` = replace objects in the bottom that **least recently used** if the maximum capacity is reached; \
   let's imagine a queue whose objects are moved to the top every time they're accessed; \
   most famous cache replacement algorithm; \
   providers a nice cache-hit rate for lots of use-cases;
```
7add   0add   1add   2add   3add  0read  4add   2read  5add
| |    | |    | |    |2|    |2|    |2|    |2|    |2|    |2|
| |    | |    |1|    |1|    |1|    |1|    [4]    |4|    |4|
| | -> |0| -> |0| -> |0| -> |0| -> [0] -> |0| -> |0| -> |0|
|7|    |7|    |7|    |7|    [3]    |3|    |3|    |3|    [5]
```
 - `MRU` = replace objects in the top that **most recently used** if the maximum capacity is reached; \
   let's imagine a queue whose objects are moved to the top every time they're accessed; \
   the algorithm seeks to preserve old data;
```
7add   0add   1add   2add   0read  3add   4add   2read  3add
| |    | |    | |    |2|    |2|    |2|    |2|    [2]    [3]
| |    | |    |1|    |1|    |1|    |1|    |1|    |1|    |1|
| | -> |0| -> |0| -> |0| -> [0] -> [3] -> [4] -> |4| -> |4|
|7|    |7|    |7|    |7|    |7|    |7|    |7|    |7|    |7|
```
 - `LFU` = keeps track how many times objects have been accessed (has a counter) and replaces the **least frequently used** objects (lowest counters); \
   problem: object with big counter value very hard to replace, although it has not been accessed for a long time
```
7add   0add   1add   2add   0read  3add   4add   2read  5add
| |    | |    | |    |2|    |2|    |2|    |2|    [2]    |2|
| |    | |    |1|    |1|    |1|    |1|    [4]    |4|    |4|
| | -> |0| -> |0| -> |0| -> [0] -> |0| -> |0| -> |0| -> |0|
|7|    |7|    |7|    |7|    |7|    [3]    |3|    |3|    [5]
```
 - `Second chance` =  modification of the FIFO algorithm; \
   each object in the cache has 1 special bit of meta-info - object previously used (yes/no); \
   if replace object have 1 bit=no, then the object is deleted otherwise bit=yes is cleared and the object is returned in the queue as a new
 - `2Q` = rule: object accessed once cannot yet be allowed to be cached; \
   the object that is accessed again may already be in the main cache. There are:
   - 2 FIFO queues: InFIFO --(evicted data)--> OutFIFO --(evicted data)--> /dev/null; OutFIFO --(accessed data)--> Main LRU; \
     OutFIFO is bigger in capacity than InFIFO
   - 1 main LRU cache: LRU --(evicted data)--> /dev/null
```
      ----------------------------
      |         LRU cache        | <---
      ----------------------------    |
                                      |
     ------------     -------------   |
 --> | In Queue | --> | Out Queue | --|
     ------------     -------------
```


Cache invalidation: \
if the data in the cache is out of date, we need to push away it
 - `TTL` = delete if the **time to live** is up; the question is how long should the TTL be?
 - `Jitter for TTL` = random TTL param helps to distribute the lifetime of data evenly by timeline
 - `Notification` = centralized update process
```
                 ---------> [Cache 1]
                 |
Notifier ---> [Broker] ---> [Cache 2]
                 |
                 ---------> [Cache 3]
```
 - `Versioning` = updating data in cache with new version of (file name, key meta-data) \
   example: caching key icon -> icon_v2 -> icon_v3
 - `Tags` = the cache is divided into different fragments, they marked with different tags; each cache fragment can be updated by tag; \
   tags: weather, currency, news, tv_programs, ...


Caches layers:
 - browser cache;
 - proxy-server cache;
 - internal application cache;
 - external service cache;
 - database cache;


## API

`CRUD` = common operations build workflow with data.

`REST` (REpresentational State Transfer) = architectural style that defines a set of constraints to be used \
for creating a distributed application in the network. This is how web services interact. \
`REST` is not a framework, a protocol or communication format. It is an architectural style or design pattern for APIs.

REST API:
- request method (GET, POST, DELETE, PUT, PATCH);
- request URI;
- request headers;
- request/response body;
- status code.
```
Request:
GET localhost:8080/books/1?param=value

Response:
200 OK
{"id":1,"title":"book 1","isbn":"0-395-08254-1","authors":[1,2,3,4],"genres":[1,2]}
```

`SOAP` = a protocol for exchanging structured XML messages between a client and a server over HTTP

`RPC` = a technology that allows a program on one server to call a function on another server as if the function were on the first server
 - gRPC (HTTP-based RPC from Google used Protobuf)
 - tRPC (HTTP-based RPC for full-stack TypeScript developers)

`GraphQL` = data query language and data manipulation language
 - one URL to communicate with server
 - the response is built based on the request body
 - resolve problem under and over fetching data for different platforms
   - under fetching = not enough response data in API endpoint (need one more endpoint)
   - over fetching = excess response data in API endpoint (extra traffic consumption)
```
request1:             response1:
{                     {
  hero {                "hero": {
    name     --->         "name": "Luke Skywalker"
  }                     }
}                     }

---
request2:             response2:
{                     {
  hero {                "hero": {
    name     --->         "name": "Luke Skywalker"
    height                "height": 1.72
    mass                  "mass": 77
  }                     }
}                     }
```

How to receive updates from the server to client?
 - `polling` = request each N-seconds and respond immediately
```
  -----------                 ----------- 
  |         | ====request===> |         | 
  | client  |                 | server  | 
  |         | <===response=== |         | 
  -----------                 ----------- 
```
 - `long polling` = request open and keep a connection; server make a response immediately if there is data 
   or later when it appears, or several answers in progress; base timeout 30 seconds
```
  -----------                 ----------- 
  |         | ====request===> |         | 
  | client  | <---response--- | server  | 
  |         | <---response--- |         | 
  -----------                 ----------- 
```
 - `webhook` = used for data transfer between source and destination systems; \
   - redirect URI: redirecting the client to another URL after processing the previous request;
   - avoid polling: callback to give delayed response on the earlier request
```
  -----------                 ----------- 
  |         | ======(1)=====> |         | 
  | order   |                 | payment | 
  | service | <-----(2)------ | service |
  -----------                 ----------- 

(1) - POST https://payment.com/api/order?callbackURL=https://order.com/confirm-payment
(2) - POST https://order.com/confirm-payment
```
 - `streaming` = a method of continuously transmitting data from a server to a client
   - server-sent events (SSE): one-way message transmission from server to client
   - websocket: thin transport layer built on top of a device's TCP/IP stack, which provides full-duplex, \
     low-latency, event-driven connections between the server and the browser


How to improve API, tips:
 - use pagination for a large number of entities sent by chunks;
 - requests and responses can be compressed;
 - caching requested data on a client side and reuse it;
 - use idempotency key to check request has been executed before (client generates and server verifies);
 - use versioning of your API (/api/v1/);



## Observability

`Monitoring` = process of collecting, analyzing, and visualizing data about an application's operation 
to ensure its reliability, performance, and availability
 - Prometheus
 - Victoria Metrics
 - Zabbix
 - Graphite

Approach to collecting metrics:
 - LTES (Google SRE)
   - latency: time to process one request
   - traffic: number of requests to the component
   - errors: number of errors
   - saturation: a quantitative metric of how much a component uses its resources
 - RED (Brendan Gregg)
   - rate: number of requests per unit of time (rps)
   - errors: number of errors
   - latency/duration: processing time of one request
 - USE (Tom Wilkie):
   - utilization: время или процент использования ресурса
   - saturation: the amount of deferred or queued "work"
   - errors: number of errors


`Alerts` = monitors changes in specific metrics using alerts and sends notifications through
a notification channel


`Logging` = event log entries: what happened, when and under what circumstances
 - errors and exceptions
 - events and user actions
 - requests to the server and third-party resources
 - information about the application status


`Tracing` = a distributed query analysis tool that can help you identify where failures occur, 
what causes poor performance, and how specific queries are being processed
 - Jaeger
 - Zipkin
 - LightStep


`Profiling` = method of analyzing and measuring software performance that provides real-time data 
on the program's operation
 - Parca
 - Pyroscope
