Fundamentals of system design
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

---

## Notes
 - defining RPS by DAU
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

 - defining traffic calculation
```
Traffic = rps * avg_request_size
---
(avg_request_size) - average request size in bytes
```
Example:

RPS = 10 000\
avg_request_size = 2Kb\
traffic = 10000 * 2 = 20 000 Kb/sec

 - defining concurrent connections (about 10% of the DAU)
```
CCU = dau * 0.1
```
Example:

DAU = 10 000 000\
CCU = 10 000 000 * 0.1 = 1 000 000
