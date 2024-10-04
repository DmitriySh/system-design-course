Homework 01: functional/non-functional requirements
=======

## System design "Social network for travelers"
![image](https://github.com/user-attachments/assets/2712c495-7088-4746-a2e3-df7fb2bdcffc)


### Functional requirements:
 - users make a new posts with photos, small description and geolocation spot of travel;
 - users can rate posts other travelers;
 - users can comment posts other travelers;
 - users can subscribe to other travelers (relations between influencers and followers);
 - users from former USSR republics
 - search popular places for traveling and viewing posts about them;
 - viewing the feed of other travelers;
 - traveler feed: 10 posts by request.


### Non-functional requirements:
 - linear growth of users for several years in a row (> 2 years)
 - geo distribution is not needed
 - user clients: web app, mobile app
 - after the year, DAU = 10 000 000
 - availability 99,9%


 - post size: 1000Kb photos + 4Kb desc + 4byte geo spot = 1 004 Kb
 - comment size: 1 Kb
 - traveler feed size: 10 * 1004Kb = 10 040 Kb
 - avg write posts per day by user: 1
 - avg write comments per day by user: 10
 - avg write reactions per day by user: 30
 - avg read feed per day by user: 5

RPS (dau * avg_requests_per_day_by_user / 86400)
 - RPS write posts: 10000000 * 1 / 86400 = 115
 - RPS write comments: 10000000 * 10 / 86 400 = 1 157
 - RPS write reactions: 10000000 * 30 / 86 400 = 3 472
 - RPS read feed: 10000000 * 5 / 86400 = 578

Traffic (rps * avg_request_size)
 - traffic write: posts + comments = 115 * 1004Kb + 1157 * 1Kb = 116 617 Kb/sec = 116 Mb/sec
 - traffic read: feed = 10 * post = 578 * 10040Kb = 5 803 120 Kb/sec = 5,8 Gb/sec

CCU (dau * 0.1)
 - connections: * 0.1 = 1 000 000 
