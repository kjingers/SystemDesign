

# Questions to Ask

* Can you give an example of how a URL shortener works?
* What is the traffic volume? (URLs generated per day / urls accessed) can make assumption of other based on read:write ratio
* How long is a shortened URL?
* What characters are allowed in a shortened URL?
* Can shortened URLs be deleted or updated?


#  Requirements

## Basic Use Cases
* URL Shortening: given long URL -> return a much shorter URL
* URL redirecting: given a shorter URL -> redirect to the original URL
* High availability, scalability, and fault tolerance

## Functional Requirements
* Given a URL, should generate a shorter and unique alias of it - called "short link". 
* When users access a short link, our service should redirect them to the original link
* Users should also optionally be able to pick a custom short link for their URL
* Links will expire after a standard default timespan. Users should be able to specify the expiration time.

## Non-Functional Requirements
* Should be highly available, because if down, url redirections won't work
* Redirection should be real-time with minimal latency
* Shortened Links should not be guessable/predictable

## Extended Requirements
* Analytics
* Our service should be accessible through REST APIs by other services

# Estimations

## Assumptions
* 100:1 read to write ratio
* 500M new URL shortening per month

## Traffic Calculations (Assumtion, Read/sec, write/sec)
* 100 * 500M = 50B redirections per month
* QPS write: 500M / (30 * 10^5) = 500 / 3 = 200 URLs / sec
* QPS Read: 200 * 100 = 20k redirections / sec

## Storage Calculations (Size assumption, storage of X years - of timeframe that makes sense)
Assume we store URL Request and Link for 5 years
* 500M * 5 year * 12 months = 500M * 60 = 30000M = 30 Billion
* Assume each object is 500 bytes: 30 Billion * 500 bytes = 15 TB 

## Bandwidth Estimates (Traffic * storage per second)
* New URLs: 200 URLs/s * 500 bytes = 100KB/s
* Redirection: 100kB * 100 = 10 MB/sec

## Memory Estimates (How much to cache)
* 80-20 rule. So want to cache 20% requests in a day
* (20k redirections / sec) * 10^5 = 2 Billion per day
* 20% * 2 bill * 500 bytes = 200 GB

# System APIs

We need two API endpoints:

**Create URL**
POST api.test.com/v1/url/shorten

Parameters:
* long-url: longURL string to shorten
* api-dev-key: for throttling
* custom-alias: optional custom alias
* expire-date: Optional Expire date

Return (string):
* short-url (or error)

**Get Long URL**
GET api.test.com/v1/{alias}

Return:
* long-url

**Optional Delete URL**
DELETE api.test.com/v1/{alias}

Return: success/error code

# Database Design: Two Tables

User table and Short <-> Long URL mapping


![image](https://user-images.githubusercontent.com/13190696/164067104-45c8ba06-c577-44ba-a8c5-3bb1df794b7d.png)

# High Level Design

![Alt text](/Drawio/urlshortener1.drawio.png?raw=true "Very High Level")


# Generating Short Unique key for given URL

## Encoding Scheme:

Can compute unqiue hash of long URL and then encoded (base36[a-z,0-9]) or base 64([A-Z,a-z,0-9,+,/])
* WIth 6 charaters and base64: 64^6 = 68.7 billio possible strings
* Base64 - each character encodes 6 bits (2^6 = 64)

## Option 1: Encoding Actual URL, or hash of url

Can compute unqiue hash of long URL and then encoded (base36[a-z,0-9]) or base 64([A-Z,a-z,0-9,+,/])
* MD5 gives 128-bit hash, requiring more than 21 characters, but we only can have 6/8. 
* Using the first 6/8 could result in duplication
* TO try to get around this, we can append some unique string to input of hash (userid, increasing sequence number). Conflict still possible.
* If duplicate, can append sequence count and retrying until get valid one
* Querying DB can be expensive when have a lot of collisions. Using Bloom filters can improve performance

![image](https://user-images.githubusercontent.com/13190696/164282721-1910dbdf-06ba-4baf-be73-98288461e769.png)

## Option 2: Base 62 encode uuid
* Requires a unide ID generator
* Collision would be impossible
* Short URL not necessarily fixed.
* Next url is predictable, which is security concern

![image](https://user-images.githubusercontent.com/13190696/164284223-51114d0e-8665-4b6a-8054-47c3a86fd4f1.png)


## Option 3: Generating Keys Offline

![image](https://user-images.githubusercontent.com/13190696/164077911-1e0ee2be-55c9-4be8-9606-46c6e1bbc9e9.png)

* Can pre generate keys and store in a database (key-DB)
* Can keep used keys in DB. ONce gnerates keys, can immediately move them into used keys
* If KGS dies before assigning all keys, some wil be lost, but that's okay since we have so many
* To not assign same key to multiple servers, but lock on data structure holding the keys before giving to server.

6 (characters per key) * 68.7B (unique keys) = 412 GB.

* Can have replicas on KGS to avoid Single Ppoint of Failure

# Other Notes
* Can partition on hash of object since we don't care about range based lookups
* Range-based partitioning could still lead to unbalanced DBs and make it hard to re-balance later. 
* If user requests expired link, then delete. 
* A seperate cleanup service can run periodically to remove expired links from storage and cache. SHould be lightweight and run when low traffic load.
* Once deleted, can put key back in key-DB.
* Private Links: Can store permission level (public/private) with each URL. For example, in Cassandra, the hash row will have columns that will store the UserIDs of those users that have permissions to see the URL.










