Homework 01: functional/non-functional requirements
=======

## System design "Social network for travelers"
![image](https://github.com/user-attachments/assets/2712c495-7088-4746-a2e3-df7fb2bdcffc)


### Functional requirements:
 - users make a new posts with photos, small description and geolocation spot of travel;
 - users can rate posts other travelers;
 - users can comment posts other travelers;
 - users can subscribe to other travelers (relations between influencers and followers);
 - search popular places for traveling and viewing posts about them;
 - viewing the feed of other travelers.


### Non-functional requirements:
 - users from former USSR republics
 - linear growth of users for several years in a row (> 2 years)
 - geo distribution is not needed
 - user clients: web app, mobile app
 - DAU: 10 000 000 (after the year)
 - availability: 99,9%
 - all requested should be averaged over the year
 - CPU, RAM, Storage peak load redundancy: +50%
 - services: reliable, scalable and fault tolerance
 - look for most popular places by reactions in travel posts


 - Limits:
   - traveler feed's posts: 10
   - photos per post: 5
   - reactions per post: 30
   - comments per post: 30


 - Timings:
   - publish post with photos: 1 sec
   - publish comment: 0,1 sec
   - publish reaction: 0,1 sec
   - view post with photos and reactions: 2 sec
   - view feed: 2 seconds
   - view post's comments and reactions: 1 sec
   - view most popular posts by reactions: 5 sec


 - Size:
   - photo size: 200Kb
   - post size: 1000Kb photos + 4Kb desc + 4byte geo spot = 1 004 Kb
   - comment size: 1 Kb
   - reaction size: 4 byte = 0,004 Kb
   - traveler feed size: 10 * 1004Kb = 10 040 Kb


 - Avg requests:
   - avg write posts per day by user: 1
   - avg write comments per day by user: 10
   - avg write reactions per day by user: 20
   - avg write photos per day by user: 1 * 5 = 5
   - avg read feed per day by user: 3
   - avg read feed's posts per day by user: 3 * 10 = 30
   - avg read comments of posts per day by user: 30 * 30 = 900
   - avg read reactions per day by user: 30 * 30 = 900
   - avg read photos per day by user: 30 * 5 = 150


 - RPS (dau * avg_requests_per_day_by_user / 86400):
   - RPS write posts: 10000000 * 1 / 86400 = 115
   - RPS write comments: 10000000 * 10 / 86400 = 1 157
   - RPS write reactions: 10000000 * 20 / 86400 = 2 314
   - RPS write photos: 10000000 * 5 / 86400 = 578
   - RPS read feed's posts: 10000000 * 30 / 86400 = 3 472
   - RPS read comments: 10000000 * 900 / 86400 = 104 166
   - RPS read reactions: 10000000 * 900 / 86400 = 104 166
   - RPS read photos: 10000000 * 150 / 86400 = 17 361


 - Traffic (rps * avg_request_size):
   - traffic write all text: posts + comments + reactions = 115 * 1004Kb + 1157 * 1Kb + 2314 * 0,004Kb = 116 626 Kb/sec = 117 Mb/sec
   - traffic write photos: 578 * 200Kb = 115 600 Kb/sec = 116 Mb/sec
   - traffic read all text: posts + comments + reactions = 3472 * 1004Kb + 104166 * 1Kb + 104166 * 0,004Kb = 3 590 470 Kb/sec = 3,6 Gb/sec
   - traffic read photos: 17361 * 200Kb = 3 472 200 Kb/sec = 3,5 Gb/sec


 - CCU (dau * 0.1)
   - connections: * 0.1 = 1 000 000 
