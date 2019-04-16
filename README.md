# Fallacies of Distributed Computing

This source is the presentation that I put together discussing how the Fallacies of Distributed Computing are still relevant.

## 8 Fallacies

    1. The network is reliable
    2. Latency is zero
    3. Bandwidth is infinite
    4. The network is secure
    5. Topology doesn't change
    6. There is one administrator
    7. Transport cost is zero
    8. The network is homogeneous

## 1. The network is reliable

The process that the executing code will always be reachable.

Things that can cause this assumption to be wrong

- Acts of God: Hurricane, Tornado, Earthquake
- Acts of Nature: Sharks
- Acts of Man: Backhoe, Human Error
- Badly behaving code

### 1a. Mitigations

Retry when operations fail, or take too long.

In the context of HTTP, Tim Bray says
> Second, the clarity about GET, PUT, and DELETE being **idempotent**, while POST isn’t, helps hugely.

- **Retry Pattern**: When a fault happens when executing an operation, classify that fault into one of three buckets: Cancel, Retry, or Retry after Delay.
- **Circuit Breaker**: Track faults against external dependencies such that the circuit can be classified as: Closed (Allow all requests), Open (Deny all requests), or Half-Open (Allow only a partial amount of requests).

### 1b. Example

![Netflix Hystrix](images/hystrix-command-flow-chart.png)

Hystrix compbines the previous three patterns into a reusable library allowing developers to more easily create fault tolerant systems.

## 2. The latency is zero

When communicating from one process to another, assuming that this call will not introduce any additional overhead.

This fallacy often comes into play when using a framework that adds a layer of abstraction. For example RPC and ORM are two common places where the developer writes code in a way that makes it appear like one line of code is locally executed, wherein it actually has to make a remote call. When this happens the liklihood of a N+1 Select problem significantly increases as the boundary between remote and local has been blurred.

With the popularity of microservices, this has also caused issues when dealing with HTTP/S connections. While the overhead of establishing an HTTP connection can seem to be quick, this is added latency to requests that can quickly add up to noticeable amounts of time. Not to mention that there is a finite limit to the nuymber of connections that can be open at a time.

### 2a. Mitigations

- **Resource Pool**: In process cache of expensive resources, such as connections to a dataaase. Allowing a resource from cache to be used instead of creating a new one each time it is needed.
- **Request Cache**: In process cache of requests to prevent duplicate requests for the same data (in a single logical operation) from being sent multiple times.
- **Request Collapsing(Batching)**: Where possible, instead of making N requests, combine those requests so that only 1 request is made.
- **HTTP/2**: Allows more effecient use of connections by allowing the server to push data down and not rely on clients to establish new connections.

When writing code, be cautious when creating functions that wrap remote calls. If doing this, please naem functions in a way so that it is obvious that a remote call is/may take place.

### 2b. Example

![Netflix Hystrix](images/hystrix-command-flow-chart.png)

Hystrix also provides the capability of a request cache, allowing requests in a single context to bypass additional requests for identical pieces of data.

## 3. Bandwidth is infinite

While networks now have improved significantly since 1994, reality is that we continue pushing more and more data across the network. A great example of this is the popularity of caching solutions such as memcache.

> memcached allows you to take memory from parts of your system where you have more than you need and make it accessible to areas where you have less than you need. [source](https://memcached.org/about)

With statements like the above, it makes it very easy to regularly transfer large objects from machine to machine. A common place for this to happen is in web applications when dealing with user state, storing such inforamtion as authentication tokens, permissions, user profile data, company profile data, etc.

### 3a. Mitigations

- Request what you need. Don't ask for 100kb of data when 3kb will do. For example if all you need from a user profile is their name, don't get the entire profile object.
- Be deliberate in how data is organized. For example XML takes up more space than JSON which is still larger than protobuf.
- It's ok to use HTTP Compression

## 4. The network is secure

Of course nobody would make this mistake and assume that the internet is a friendly place. But what about the network that connects the ingress point (web server) to the back end services (databases, etc.)? A backend service that stores things, such as credit card numbers, should make every assumption that it will be under direct attack.

### 4a. Mitigations

- Classify data and capabilities
- Authenticate and Authorize remote calls
- Encrypt in transit and at rest
- Utilize a service mesh

### 4b. Examples

- Istio

## Sources

- [Fallacies of Distributed Computing Explained, by Arnon Rotem-Gal-Oz](http://www.rgoarchitects.com/Files/fallacies.pdf)
- [Lessons learned from teaching Distributed Systems for 15 years](https://kriha.de/DistributedSystemsLectures.html)
- [Tim Bray: How Relevant Are The Fallacies Of Distributed Computing Today?](http://www.tbray.org/ongoing/When/200x/2009/05/25/HTTP-and-the-Fallacies-of-Distributed-Computing)
- [9th Fallacy of Distributed Computing: Location is irrelevant](http://ecbeez.blogspot.com/2010/10/9th-fallacy-of-distributed-computing.html)
- [The '8 Fallacies of Distributed Computing' Aren't Fallacies Anymore](https://www.stackery.io/blog/eight-fallacies/)
- [The Fallacies of Distributed Computing Reborn: The Cloud Era](https://blog.newrelic.com/engineering/the-fallacies-of-distributed-computing-reborn-the-cloud-era/)

1. The network is reliable
    - [Retry Policy](https://docs.microsoft.com/en-us/azure/architecture/patterns/retry)
    - [Microsoft: Circuit Breaker](https://docs.microsoft.com/en-us/azure/architecture/patterns/circuit-breaker)
    - [Fowler: Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
    - [Netflix: Hystrix](https://github.com/Netflix/Hystrix/wiki/How-it-Works)
2. Latency is zero
    - [Fowler: Resource Pool](https://martinfowler.com/bliki/ResourcePool.html)
    - [N+1 Select Problems](https://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem-in-orm-object-relational-mapping)
3. Bandwidth is Infinite
4. Network is secure
   - [What is Data Classification?](https://digitalguardian.com/blog/what-data-classification-data-classification-definition)
   - [What is Istio](https://istio.io/docs/concepts/what-is-istio/)
   - [Nginx: What is a Service Mesh](https://www.nginx.com/blog/what-is-a-service-mesh/)
