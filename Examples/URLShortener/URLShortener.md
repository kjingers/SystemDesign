

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
long-url: longURL string to shorten
api-dev-key: for throttling
custom-alias: optional custom alias
expire-date: Optional Expire date

Return (string):
short-url (or error)

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

![Alt text](/SystemDesign/blob/main/Drawio/urlshortener1.drawio.png?raw=true "Optional Title")



# High Level Design










