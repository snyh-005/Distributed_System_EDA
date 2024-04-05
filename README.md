# Distributed System Design

This is a diagram of a sample distributed system design using azure and other middlewares that are commonly used.

Disclaimer: For any system design based on the problem statement and trade offs the placement and the middleware will change.

This diagram can be divided into  7 sections

1. Client - Network
2. API Gateway
3. Microservices
4. Database Caching
5. Database
6. Azure Container Registry
7. Observability tool


## Client- Network

  1. The customer (actor) will perform an action.
  2. The application (web / mobile) access the domain name such as www.google.com
  3. DNS is a paid service provided by 3rd party (Amazon Route 53 or Azure DNS)
  4. DNS returns the IP address to the application (web / mobile)
  5. Once IP address is obtained, HTTP/S requests are sent to webserver.

    a) It all starts when you (the DNS client) type or paste the domain name of a website into an address bar on a search engine. Because computers don’t use human language, this domain name has to be converted into an IP address in order for your computer to find it. So the computer immediately sends your request to a DNS recursive resolver for translation.
    b) First, the resolver goes to the root nameserver (FYI, some sources spell it with two words: “name server”) that aligns with the website’s top-level domain (TLD). The TLD is indicated by the domain extension — .com, .org, etc. Root nameservers keep lists of which authoritative nameservers align with which TLDs. 
    c) Now that it knows where to go, the resolver visits the authoritative TLD nameserver that corresponds with that extension and begins looking for the specific IP address. 
    d) Once it tracks the IP address down, the resolver passes it to the authoritative nameserver to be authenticated. 
    e) The authoritative nameserver checks the IP address. Once it gets the expected response, it knows it’s found the website you’re looking for. 
    f) The authoritative nameserver sends the confirmed IP address back to the web browser where this search first began. As soon as your web browser receives the IP address, the process is complete and the intended website will appear!

  6. The request that reached the webserver is first passed through the **Azure Front Door**

    It acts as a single entry point for your web applications, providing features like:
    a) Intelligent routing: Directs users to the closest healthy origin server (e.g., an Azure web app) based on factors like latency. 
    b) Web Application Firewall (WAF): Protects your applications from malicious attacks. 
    c) Load balancing: Distributes traffic across multiple origin servers for better performance and scalability.

## API Gateway
    
In a microservice architecture pattern, services are decomposed based on domain, business etc
API Gateway is another service user to perform any logic other than business. It's the single point of entry.

*Note:*  Entry to the API Gateway can be handled by RabbitMQ 
        
    1. Proxying
    2. Authenticate the request
    3. Authorize the request
    4. Rate Limiter and Security
    5. Input Validation
    6. Monitoring and Logging
    7. Cache Metrics Collection
    8. Routing
    9. Circuit Breaker
    10. Load Balancing

## Microservices

The routed requests from the API Gateway is then sent to each service instances.
The instances are deployed on Kubernetes which is mounted either on Azure or AWS.
The EKS or AKS routes the requests to a service instance.

### Scalability
* EKS: Requires more manual configuration for auto-scaling. You can implement auto-scaling policies using tools like AWS Auto Scaling, but it's not built-in like AKS. EKS offers a maximum of 100 nodes per cluster per account.

* AKS: Provides built-in Cluster Autoscaler for automatic scaling based on workload. This simplifies scaling for short-lived processes. AKS allows up to 500 nodes per cluster.

### Availability
* EKS: Manages the availability of the Kubernetes control plane automatically. This ensures core cluster functionality remains operational even if a control plane node fails.

* AKS: Also manages control plane availability. Additionally, AKS offers availability zones for worker nodes. Distributing worker nodes across zones improves fault tolerance as an outage in one zone won't affect the entire cluster.

| Feature	                        | EKS                       | AKS                           |
|---------------------------------|---------------------------|-------------------------------|
| Auto-scaling	                   | Manual configuration      |  Built-in Cluster Autoscaler  |
| Maximum Nodes per Cluster       | 100                       | 500                           |
| Control Plane                   | Availability              | Managed                       |
| Worker Node Availability Zones  | Not Built-in              | Supported                     |  


## Cache

### Application Cache and Database cache

Cache could be Embedded cache or Shared Cache. 

* Embedded Cache: It's a private cache within the application (eg: Spring boot starter cache)

* Shared Cache: It's the external cached shared between multiple servers (eg: Redis )

---------------------------------------------------------------------

* Distributed Cache:  A shared cache spread across multiple servers for high availability and scalability. (eg: Redis)

*Note: Redis itself isn't inherently a distributed system. While Redis offers features for coordination, it doesn't handle complex distributed system functionalities like automatic failover or data consistency guarantees (refer to CAP theorem). 
These aspects might require additional tools or frameworks. In distributed system load on Redis is distributed (having each redis for each service or domain) or shrading.*

* Reverse Proxy Cache:  A cache sits in front of the microservices, handling static content and API responses. (eg: ngnix)


## Database

##### Database replication

it's usually with a master/slave relationship between the original (master) and the copies (slave).

### Non-relational DB

1. Application requires super-low latency
2. Data are unstructured
3. You only need to de-serialize and serialize the data
4. Store massive amount of data

### Relational DB

1. When high scalability is required
2. Data is structured and has relations
3. We need to perform heavy computing operations.
4. When complex queries needs to be written

#### Scalability

* Vertical Scaling / Scale Up, means the process of adding more power (CPU, RAM etc) to your servers.
  * It is impossible to add unlimited CPU and memory.
  * Does not have failover and redundancy.
* Horizontal Scaling / Scale Out, allows you to scale by adding more servers into your pool of resources.

  ##### Sharding:
  A core concept in horizontal scaling is sharding. Data is divided into smaller, more manageable chunks based on a shard key (e.g., user ID, product category). Each shard is then stored on a separate server in the cluster. 
  
  *Benefits:*

  * Distributes the load for reads and writes across multiple servers. 
  * Enables adding more servers to handle increased data volume or traffic. 
  
  *Challenges:*

  * Requires careful shard key selection to ensure even distribution of data and avoid bottlenecks. 
  * Complex queries that span multiple shards might require additional processing.

  
#### Availability

##### Replication:
Another technique used alongside sharding is replication. Each shard has one or more replicas on different servers. 

 *Benefits:*

* Improves data availability. If a server fails, a replica can take over, minimizing downtime.
* Enables read scaling by directing read requests to replicas, reducing load on the primary shard.

  *Considerations:*

* Requires data synchronization between replicas to maintain consistency. Different replication models offer varying trade-offs between consistency and performance.
