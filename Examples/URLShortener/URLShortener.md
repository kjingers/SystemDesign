

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

