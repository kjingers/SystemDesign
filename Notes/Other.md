# Common Architecture to a lot of distributed system designs

https://www.youtube.com/watch?v=iJLL-KPqBpM

![image](https://user-images.githubusercontent.com/13190696/164074824-76701bbd-0afe-4919-b046-4f14bacc1d07.png)

# Load Balancer, handling increased throughput

![image](https://user-images.githubusercontent.com/13190696/164073105-83af078a-2c85-4752-8a49-035ee2aac2d3.png)

To handle scalability, can have domain resolve to many load balancers. The IP addresses of load balancer can be put in DNS. 
Can also put different load balancers in different datacenters to improve availabiliity.

# FrontEnd Web Service

![image](https://user-images.githubusercontent.com/13190696/164075140-d1b63fae-d595-469a-8f6f-225549d700d6.png)

# Metadata Service

![image](https://user-images.githubusercontent.com/13190696/164075405-e1f56396-bb4c-48f2-8063-7f5d23da6215.png)

# Security
Cna use SSL over HTTP between client and FE WEbservice, and encryption beyond that.


