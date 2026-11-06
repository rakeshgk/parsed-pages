# Chapter 02 - Defining Nonfunctional Requirements

If you are building an application, you will be driven by a list of requirements. At the top of your list is most likely the functionality that the application must offer : what screens and buttons you need, and what each oepration is supposed to do in order to fulfill the purpose of your software. These are functional requirements. In addition, you have nonfunctional requirements. These requirements might not be explicitly written down, because they may seem somewhat obvious, but they are just as important as the app's functionality.

## Describing Performance

Most discussions of software performance consider two main types of metric:

1. **Response Time** - The elapsed time from the moment when a user makes a request until they receive the requested answer. The unit of measurement is seconds (or milliseconds, or microseconds).
2. **Throughput** - The number of requests per second, or the data volume per second, that the system is processing. For a given allocation of hardware resources, there is a maximum throughput that can be handled. The unit of measurement is “somethings per second.”

Throughput and Response Times are often related. The service has a low response time when request throughput is low, but response time increases as load increases. This is because of queuing: when a request arrives on a highly loaded system, the CPU is likely already in the process of handling an earlier request, and therefore the incoming request needs to wait until the earlier request has been completed. As throughput approaches the maximum that the hardware can handle, queuing delays increase sharply.

If a system is close to overload, with throughput pushed close to the limit, it can sometimes enter a vicious cycle where it becomes less efficient and hence even more overloaded. Long queue of requests may cause client timeouts making them resend their requests. This causes the rate of requests to increase even further, making the problem worse—a retry storm. Even when the load is reduced again, such a system may remain in an overloaded state until it is rebooted or otherwise reset. This phenomenon is called a metastable failure, and it can cause serious outages in production systems. 

To avoid retries overloading a service, you can increase and randomize the time between successive retries on the client side (exponential backoff) and temporarily stop sending requests to a service that has returned errors or timed out recently (by using a circuit breaker or token bucket algorithm). The server can also detect when it is approaching overload and start proactively rejecting requests (load shedding) or send back responses asking clients to slow down (backpressure). 
