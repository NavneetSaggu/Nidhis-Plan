# Introduction

## **What Is Hystrix?**

- Hystrix helps control our interactions with dependent services by adding
    - Latency Tolerance logic
    - Fault Tolerance logic
- It does this by
    - Isolating points of access between the services
    - Stopping cascading failures across them
    - Providing fallback options
    
    all of which improve your system’s overall resiliency.
    

### Resources

The following links provide more context around Hystrix and the challenges that it attempts to address:

- [“Making Netflix API More Resilient”](http://techblog.netflix.com/2011/12/making-netflix-api-more-resilient.html)
- [“Fault Tolerance in a High Volume, Distributed System”](http://techblog.netflix.com/2012/02/fault-tolerance-in-high-volume.html)
- [“Performance and Fault Tolerance for the Netflix API”](https://speakerdeck.com/benjchristensen/performance-and-fault-tolerance-for-the-netflix-api-august-2012)
- [“Application Resilience in a Service-oriented Architecture”](http://programming.oreilly.com/2013/06/application-resilience-in-a-service-oriented-architecture.html)
- [“Application Resilience Engineering & Operations at Netflix”] ([https://speakerdeck.com/benjchristensen/application-resilience-engineering-and-operations-at-netflix](https://speakerdeck.com/benjchristensen/application-resilience-engineering-and-operations-at-netflix))

## **What Is Hystrix For?**

Hystrix is designed to do the following:

- Give *protection* and *control* over *latency* and *failure* from dependencies accessed (typically over the network, could also be within service) via third-party client libraries.
- Stop cascading failures in a complex distributed system.
- Fail fast and rapidly recover.
- Fallback and gracefully degrade when possible.
- Enable near real-time monitoring, alerting, and operational control.

## **What Problem Does Hystrix Solve?**

In distributed architectures, dependencies will inevitably fail at some point. 

If the host application is not isolated, it can fail because of these failures too.

- Math for an example
    
    > For example, for an application that depends on 30 services where each service has 99.99% uptime, here is what you can expect:
    99.99*30 = 99.7% uptime
    0.3% of 1 billion requests = 3,000,000 failures
    2+ hours downtime/month even if all dependencies have excellent uptime.
    Reality is generally worse.
    Even when all dependencies perform well the aggregate impact of even 0.01% downtime on each of dozens of services equates to potentially hours a month of downtime **if you do not engineer the whole system for resilience**.
    > 

### Healthy

![](https://github.com/Netflix/Hystrix/wiki/images/soa-1-640.png)

### One Dependency Down

![](https://github.com/Netflix/Hystrix/wiki/images/soa-2-640.png)

Every point in an application that makes network requests is a source of potential failure. 

Worse than failures, can be latency, which backs up queues, threads, and other system resources causing even more cascading failures across the system.

![](https://github.com/Netflix/Hystrix/wiki/images/soa-3-640.png)

All of these represent failure and latency that needs to be isolated and managed so that a single failing dependency can’t take down an entire application or system.

## What Design Principles Underlie Hystrix?

Hystrix works by:

- Preventing any single dependency from using up all container (such as Tomcat) user threads.
- Shedding load and failing fast instead of queueing.
- Providing fallbacks wherever feasible to protect users from failure.
- Using isolation techniques (such as [bulkhead](Bulkhead.md), swimlane, and circuit breaker patterns) to limit the impact of any one dependency.
- Optimizing time-to-discovery through near real-time metrics, monitoring, and alerting
- Optimizing time-to-recovery by
    - low latency propagation of configuration changes
    - support for dynamic property changes
    
    which allows you to make real-time operational modifications with low latency feedback loops.
    
- Protecting against failures in the entire dependency client execution, not just in the network traffic.

## **How Does Hystrix Accomplish Its Goals?**

Hystrix does this by:

- Wrapping all calls to external systems (or “dependencies”) in a `HystrixCommand` or `HystrixObservableCommand` object which typically executes within a separate thread (this is an example of the [command pattern](http://en.wikipedia.org/wiki/Command_pattern)).
- Timing-out calls that take longer than thresholds you define. 
Slightly higher than 99.5th percentile latency for each dependency, for eg.
- Maintaining a small thread-pool (or [semaphore](Bulkhead.md)) for each dependency; if full, requests are rejected instead of queueing.
- Measuring successes, failures (exceptions thrown by client), timeouts, and thread rejections.
- Tripping a circuit-breaker; stop requests to a service for a period of time, either manually or automatically if the error percentage for the service passes a threshold.
- Performing fallback logic when a request fails, is rejected, times-out, or short-circuits.
- Monitoring metrics and configuration changes in near real-time.

When wrapping in Hystrix, architecture looks like this diagram.

1. Each dependency is isolated from one other; restricted in the resources it can saturate when latency occurs
2. Calls are covered in fallback logic that decides what response to make when any type of failure occurs in the dependency

![](https://github.com/Netflix/Hystrix/wiki/images/soa-4-isolation-640.png)

Learn more about [How It Works](https://github.com/Netflix/Hystrix/wiki/How-it-Works) and [How To Use](https://github.com/Netflix/Hystrix/wiki/How-To-Use).