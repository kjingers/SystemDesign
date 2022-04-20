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
* Lets go with NoSQL
* We can use the above schema in distributed key-value store (like DynamoDB). For Photo table, key would be PhotoID.
* If we have post comments and post likes in seperate tables, we could use Cassandra to support many attributes 





