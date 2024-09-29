01 - Fundamentals of system design
=======

## Criteria of information systems
- `Latency` = time spent on servicing one customer
- `Low-latency` = seamless and realtime with fast data flow
```
   client -(t1)-> server -(t2)->
                                 database
   client <-(t4)- server <-(t3)-
```
- `Throughput` = the number of processed customer requests per unit of time
- `High-throughput` = high link utilization
```
   client -(200 req)-> server -(100 req)-> database
```

 - `Data-intensive applications` = requirements for:
   - significant storage capacity;
   - efficient data transfer rates;
   - ability to search in large amounts of data.


 - `Compute-intensive applications` = requirements for substantial processing power


 - `Read-intensive applications` = ?
 - `Write-intensive applications` = ?

---

## Properties of information systems
 - `Scalability` = the ability of the system growing: \
   without loss of performance and other characteristics, \
   without the need to change the software implementation.
   - scale up = adding more compute resources to an existing system (CPU, RAM, SSD);
   - scale out = distribute the workload across different servers.


 - `Load parameters`
   - requests per second (RPS);
   - traffic (Kb/Mb/Gb/Tb per second);
   - number of concurrent connections/users (CCU);
   - number of daily/monthly active users (DAU/MAU);


 - `Fault tolerance` = the ability of the system to continue to function correctly
   in the presence of component failures:
   - deploying new code;
   - migrating data;
   - changing configuration files;
   - network or disk problem;
   - higher users workload;
   - accidents in the data center;
   - ... (others)


 - `Ensuring fault tolerance`: 
   - redundancy at the hardware level;
   - software-level failure detection and handling;
   - predicting and preventing failure.



 - `Availability`: users interact with the system 
   and receive the expected responses in acceptable time frame; 
   accessibility is measured in the number of nines
```
Available   Downtime/y      Downtime/m      Downtime/w      Downtime/d
----------------------------------------------------------------------
99%      -  3,65  days   -  7,31  hours  -  1,68  hours  -  14,40 min
99,5%    -  1,83  days   -  3,65  hours  -  50,40 min    -  7,20  min
99,8%    -  17,53 hours  -  87,66 min    -  20,16 min    -  2,88  min
99,9%    -  8,77  hours  -  43,83 min    -  10,06 min    -  1,44  min
99,95%   -  4,38  hours  -  21,92 min    -  5,04  min    -  43,20 sec
99,99%   -  52,60 min    -  4,38  min    -  1,01  min    -  8,64  sec
99,995%  -  26,30 min    -  2,19  min    -  30,24 sec    -  4,32  sec
99,999%  -  5,26  min    -  26,30 sec    -  6,05  sec    -  864   ms
```
 - `Reliability` = the ability to work without failures or errors
  for a long time


 - `MTBF` (mean time between failure) - average time without failure;\
   it is a unit of reliability measurement for recoverable systems;\
   the more time passes between failures, the more reliable the system is.

Example:

period = 24 hours\
lost time = 2 hours\
issue count = 2\
MTBF = (24 - 2) / 2 = 11 hours

 - `Convenience of maintenance` = requirements for:
   - updating the system without downtime;
   - ability switch off the part of services;
   - availability of system monitoring and detailed logging;
   - time between detect the problem and fix it;
   - possibility of expanding the system.

 - `Safety` = requirements for:
   - prevention of possible threats (data leak, data falsification);
   - protection against attacks (DDoS)

---

## Load balancing
This is a method of distributing network traffic across a pool of servers.
 - `Client balancing` = client knows the list of servers and distributes requests to them directly
```
         ----> server 1
         |
client ------> server 2
         |
         ----> server 3
```

 - `Server balancing` = client doesn't know the list of servers serving the request, the `LoadBalancer` server does it instead, \
   it uses health checks to route direct traffic only to "live" servers. 
```
             ------> server 1 (Ok)
             |
client ---> [LB] --> server 2 (Ok)
             |
             ---X--> server 3 (Down)
```

 - Algorithms:
    - `Random` = incoming requests are randomly assigned to servers within the pool.
    - `Round robin` = distribution of incoming requests across the server pool sequentially, \
      starting from the first server and ending with the last (<ins>default</ins> for the most cases).
    - `Weighted round robin` = each server in the pool assigns `weights` based on their capabilities, \
      incoming requests are distributed proportionally to these weights, ensuring that more powerful servers \
      handle a larger share of the workload (<ins>for large-scale infrastructures</ins>)
    - `Least connections` = distribution of incoming requests to the server with the fewest active connections;\
      the goal is to evenly distribute the workload among servers, preventing and single node from becoming overloaded;\
      (<ins>for long-lived connections</ins>)
    - `Least response time` = when distributing incoming requests, priority is given to servers \
      with the shortest response time and the minimum number of active connections; \
      it enhances user experience and optimizes resource utilization.
    - `Sticky sessions` = also known as `session affinity`, requests from a specific client are always routed to the same server, \
      it can be done, for example, using a hash index by clientId or source and/or destination IP address; \
      (<ins>for storing session data in server memory</ins>)
    - `Power of two choises` = incoming requests are choose between two randomly selected servers based on `Least connections`;\
      you don’t have to compare all servers to choose the best option each time; instead, you only need to compare two


 - Layers:
   - `Layer 4` = operating at the `Transport Level OSI model`, using the TCP and UDP protocols, \
     manages traffic based on network information such as application ports and protocols \
     without visibility into the actual content of messages.\
     Layer 4 effective approach for simple packet-level load balancing.
   - `Layer 7` = operating at the `Application Level OSI model`, using protocols such as HTTP and SMTP \
     to make more intelligent routing decisions based on the headers, actual content of each message, URL type, and cookie data.\
     Layer 7 can be used for the "sticky sessions" algorithm.


 - `DNS balancing` = advanced technique for distributing incoming traffic across multiple servers\
   and keeping your product running smoothly even if some of the load balancers fall
```
             -------> [LB1: Ok] ----------> server 1
             |                       |
client ---> [DNS] --> [LB2: Ok] ----------> server 2
             |                       |
             ---X---> [LB3: Down] --------> server 3
```

 - `geoDNS balancing` = redistributes application traffic across data centers in different locations \
   for maximum efficiency and security based on the geographic location of the user.
```
client: USA       -------> [LB: USA] ----------> server 1
  |               |            |     
  |----------> [geoDNS]        ----------------> server 2
  |               |                 
client: India     -------> [LB: India] --------> server 3
                               |
                               ----------------> server 4
```

 - `Service discovery` =  is the act of keeping track of the list of services that are available,\
   where those services are located on the network, and determines which servers are available and how to get to them.\
   This information is transmitted to the load balancer to update the list of available servers
```
            [SD] <--- server 1 (Down)
             |
             ⌵
client ---> [LB] ---> server 2 (Ok)
             |
             -------> server 3 (Ok)
```

---

## Proxying
Proxy server is a special server that sits between service 1 and service 2 for acting as an intermediary for tasks:
 - access protection
 - data caching
 - traffic restriction:
   - leaky bucket algorithm,
   - rate limiter algorithm,
 - data compression and modification
 - circumventing access restrictions
```
server 1 --> [proxy] --> server 2
```

Types:
 - `Forward proxy` = is an intermediary server that <ins>forwards</ins> client outbound requests to other servers:
   - masking the client's IP address for privacy
   - managing access to websites or services
   - caching frequently requested content
   - examples: Nginx, Squid
```
                              |
client --> [forward proxy] ---|--> {internet} --> server
                              |
```
 - `Reverse proxy` = handles incoming traffic, distributing it to one or more backend servers:
   - load balancing client requests across multiple servers
   - caching frequently requested content
   - protecting servers from direct attacks from the Internet
   - handling SSL encryption and decryption
   - managing server updates and maintenance
   - examples: Nginx, HAProxy, Apache, Traefik
```
                                     ------------> server 1
                         |           |
client --> {internet} ---|--> [reverse proxy] ---> server 2
                         |           |
                                     ------------> server 3
```

---

## Functional/Non-functional requirements
 - `Functional requirements` = possibilities that the system should perform to meet the needs of the user\
Examples:
   - user registration;
   - authentication and authorization;
   - multi-language support;
   - product search and filtering;
   - creating and editing product cards;
   - sending notifications;
   - integration with payment systems;
   - creating reports.


 - `Non-functional requirements` = properties that the system should have that are not related to it behavior of the system\
Examples:
   - количество обслуживаемых пользователей (100 million);
   - number of active users (MAU=20 million, DAU=670 thousand);
   - read/write user action ratio (10/1 => 10% write to the database => 67 thousand users);
   - data retention period in the database (10 years);
   - storage size for all users (document=1 mb => 232 Tb of data for 10 years);
   - worldwide availability (geo-distributed DNS cluster);
   - response time to the request;
   - data security and protection;
   - the ability of the system to cope with an increase in load;
   - maintainability.

---

## Notes
 - Defining RPS by DAU
```
RPS = dau * avg_requests_per_day_by_user / 86400
---
(avg_requests_per_day_by_user) - the average number of user's requests in one day
(86400) - 24 hours * 60 minutes * 60 seconds
```
Example:

DAU = 10 000 000\
avg_requests_per_day_by_user = 5\
RPS = 10000000 * 5 / 86400 = 578

 - Defining traffic calculation
```
Traffic = rps * avg_request_size
---
(avg_request_size) - average request size in bytes
```
Example:

RPS = 10 000\
avg_request_size = 2Kb\
traffic = 10000 * 2 = 20 000 Kb/sec

 - Defining concurrent connections (about 10% of the DAU)
```
CCU = dau * 0.1
```
Example:

DAU = 10 000 000\
CCU = 10 000 000 * 0.1 = 1 000 000
