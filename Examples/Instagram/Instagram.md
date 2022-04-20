This design does not do super deep dive into newsfeed generation. But must support it.

Good design which has added features like: comments, likes, news feed generation details, all APIs, etc

# Questions to Ask

* Is this a mobile app? Web app? Both?
* What are the important features?
* Do users follow other users? Or is it like a friends system?
* What is the traffic volume?
* Can users post images and videos? Any other types?
* Do users need the ability to search based on photo/video titles?

# Functional Requirements
* Users should be able to upload/download/view photos
* Users can perform searches based on photo/video titles
* Users can follow other users
* The system should generate and display a user's News Feed consisting of top photos from all the people the user follows

* We are not focusing on tags, like, comments, stories, etc

# Non-functional Requirements
* Service needs to be highly available
* Acceptable latency is 200ms for News Feed Generation
* Consistency can take a hit in the interest of availability if a user doesn't see a photo for a while; it should be fine
* System should be highly reliable; any uploaded photo or video should never be lost.

# Capacity Estimation

## Assumptions / Throughput
* Assume there is some API to upload photo and API to load photo individually or collection
* Assume 500M total users
* Assume 1M DAU
* Assume 2M new photos every day -> ~25 photos per second

## Storage
* Assume 200KB average photo size
* Space for 1 day: 2M * 200KB = 400 GB
* Space for 10 Years: 400GB * 365 * 10 = 4TB * 400 = ~1600 TB
* NOTE: If we support videos, then should Assume video uploads per day. Can assume 50MB per video

# Bandwidth
* 25 photos per second * 200KB = ~8MB / sec
* Assuming Read:Write of 80:20, Read = ~40 MB / sec

# APIs
Upload Photo - PhotoID uploadPhoto(UserID, Base 64 Encoded Photo)  : /photo/upload

Follow a User - Result followUser(UserID, FollowerID) : /user/follow

Unfollow a User -Result unFollowUser(UserID, FollowerID) : /user/unfollow

User New Feed - Post[] newsFeed(UserID) : /user/feed

# High Level Design

From here, we can talk about high-level design including:
* High level diagram satisfying use cases / function requirement
* Data model / Database schema
* Component Design

For high level design, can just show simple block diagram to start. But it might be better to show reads/writes seperated. In real interview, may need to focus more on news feed generation and using microservices:

![image](https://user-images.githubusercontent.com/13190696/164294274-a9fe1dec-5710-4fe4-b66e-c8b2dca840c4.png)

![image](https://user-images.githubusercontent.com/13190696/164294309-b57bcebf-f1a4-4900-91ff-740cd1b7f6de.png)

# Database Schema

For our simple design, we need minimum to keep track of UserInfo, UserFollow, and PhotoMetadata.

![Screen Shot 2022-04-20 at 2 06 54 PM](https://user-images.githubusercontent.com/13190696/164294826-013f996f-49ca-4a2d-ad20-a6ad61fe67c9.png)

* Can consider both SQL and NoSQL options. We do require joins. But SQL do have scalability concerns and we do have to support a very large userbase.
* Instagram really uses PostgresSQL
* We don't necessarily need ACID compliance. But also don't see schema chaning too much
* Lets go with NoSQL
* We can use the above schema in distributed key-value store (like DynamoDB). For Photo table, key would be PhotoID.
* If we had likes and comments, each should be in wseperate tables
* If we have post comments and post likes in seperate tables, we could use Cassandra to support many attributes?
* Can consider Graph DB for User Follows table (Neo4J)
* Amazon S3 or HDFS for photo objects themselves

# Component Design

Since writes (posts) can be slow, we should definitely seperate reads and writes so that writes don't hold up reads. (Since servers have a connection limit). So, we can have different servers for writes and different servers for reads. This allows us to optimize them independently.

Again, since this design does not focus on timeline generation and use of microservices, this is all we cover. 

# Scaling the Design

# Reliability and Redundancy
* Multiple upload/download services
* Replicate Image Store. Can have redundant secondary store, that can take control after failover
* Replicate / Backup Image metadata too

# Data Sharding

## Can partition based on UserID
* To generate unique PhotoIDs, each DB shard can have its own atuo-increment sequence. 
* Example: PhotoID = UserID % 10 + Shard Seq #
* How uses would fill certain shards and route a lot of traffic to certain shards
* Some users may have a lot more photos than others
* If shard acts up/ is down, then some user's ohotos will be completely unavailable

## Can partition based on PhotoID
* WOuld need to generate unique photoID before assigning to shard
* One option: Dedicate separate BD instance to generate auto-incementing IDs. Can define a table containing only a 64-bit ID field. Whenever we would add a photo, we can insert a new row in this table and take the ID to be our PhotoID. But this is a single point of failure.
* To avoid Single Point of failure, can have 2 or more such servers. If 2 servers, one increments even and one increments odd. Put load balancer in front to handle downtime. We could use this method using different tables to get IDs for users, comments, etc.
* Another option is to use key generation service like in TinyURL design

* For future grown, we should start with many logical partitions. Then, assign logical partitions to noew nodes as needed.

# Ranking and News Feed Generation
* If we were to pull all info on read, the latency would be too high. (Find all followers, get follower's latest photos, merge, sort, send to ranking service)

## Pre-generating the News Feed
* Dedicated servers that are continously generating users' News Feeds and storing them in a UserNewsFeed Table.
* When News Feed is requested, pull from this table noting the time this feed was generated, then pull posts newer than this time.

## Sending News feed to users
* Clients can pull news-feed. Downside, updates will not show until pulled. And will result in a lot of empty responses
* Servers can Push news feed (long Poll). Downside is user who follows a lot of accounts, and celebritys with a lot of followers. 
* Hybrid approach is best. Celebritie posts have pull-based model. Everyone else push.

## News Feed Creation with Sharded Data
* Requirement is to create news feed based on followers' latest photos. 
* Easyiest to make time creation part of the photoID, since we will already have primary index on photoID.
* More on this in Designing Twitter

## Cache and Load balancing
* Should cache metadata rows
* Possibly use CDNs to push content geographically closer to users



