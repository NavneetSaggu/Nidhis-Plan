# Istio

Below are main components of Istio Control Plane

- Istiod
    - Pilot - *Service Discovery, Config propagation*
    - Citadel - *Provides keys & certificates*
    - Galley - *Configuration validation & translation*
- Istio CNI - *Configure iptables rules to intercept traffic*

Istio is a service mesh. A service mesh uses sidecar pattern where all inbound and outbound communication of a service with others takes place through the sidecar proxies. Together, these sidecars form a mesh network.

It works by adding an Envoy sidecar to all our apps. All traffic goes through this sidecar, and the sidecar helps with

1. Monitoring all L7 layer traffic (Application Layer) [Web Dev / Internet / Networks](../Web%20Dev%20Internet%20Networks/Web%20Dev%20Internet%20Networks.md) 
2. Distributed tracing with all integrated microservices
3. Abstracting functions unrelated to application business logic into a common infrastructure reduces the complexity of code. This includes functions related to
    1. Observability (Metrics and such)
    2. Traffic Management
    3. Security (SSL Decryption and things)
4. Reducing code duplication and effort in integrating the third-party libraries like Hysterix, etc in all the applications, since rate limiting / circuit breaking can be abstracted to Isto. 
(*DOUBT: How will we configure threadpools without Hystrix? Will will only use Hystrix for threadpools in that case?*)
5. Sidecars can be upgraded independent of application code.

## Capabilities Provided By Istio

- Secure service-to-service communication in a cluster with TLS encryption, strong identity-based authentication and authorization
- Automatic load balancing for HTTP, gRPC, WebSocket, and TCP traffic
- Fine-grained control of traffic behavior with rich routing rules, retries, failovers, and fault injection
- A pluggable policy layer and configuration API supporting access controls, rate limits and quotas
- Automatic metrics, logs, and traces for all traffic within a cluster, including cluster ingress and egress

## Intro: Brief On Kubernetes

- Solves for deployment complexities
- Features
    - Declarative deployment of applications, we only declare the spec we want to achieve and k8s does the rest
    - Standardised interfaces to spec various kinds of deployment topologies (kubectl)
    - Well defined interfaces to provide additional functionalities with CRDs (**customresourcedefinitions** can be made on the same API server), controllers etc.
    - Out of the box availability of ‘Service’ construct i.e. s complete bumdle of replcasets, their pods, their deployment strategies, and the ELBs the pods sit behind.

## Intro: Why Use Istio? Problem Statement

- Number of microservices are growing
- Nature of interaction is becoming complex.
- No high level view of service interaction
- Every service developer needs to solve for
    - Connectivity - connecting to other services
    - Deployment - deployment of application
    - Security - securing access to APIs
    - Observability - MTL (metrics, telemetry, logging) for the application
    - Traffic control - L7 controls for application - path based routing etc;
    - Fault tolerance - Tolerate failures at multiple levels. Failures at node level can be solved at K8, but failure at DC level can’t be solved by K8s. Solution of these failures is needed for Helios readiness.

## What Is A Service Mesh?

- Service Mesh
    - A software infrastructure layer for controlling and monitoring internal, service-to-service traffic in microservices applications
- Core features offered
    - Connectivity
    - Control
    - Security
    - Observability

## How can service mesh be implemented?

![Untitled](Istio/Untitled.png)

### Library

You can abstract these things to a common library, which was written in all the languages so that they can be used across different applications. (Different Colors → Different Language)

These apps will be integrated with these libraries.

Cons

- It’s not language agnostic, will need to write library in each app.
- It will need redeployment in case of uipdates to the library

Pros

- It doesn’t need addition of another proxy

### DaemonSet

A proxy is used for each node. The proxy will have to be multi-tenant to be able to function for all the different kinds of apps it supports.

Cons

- The proxy needs to be multi-tenant, and needs to be aware of the applications it’s handling in the node.
- Any performance issue happening in one app can affect the other apps who’s traffic passes through the proxy, since this is multi-tenant.

### Sidecar

Cons

- This is resource heavy

Pros

- It doesn’t need any handling of multi-tenancy since each proxy sidecar is seperated for each application

## Why Istio?

- Need for service mesh is a generic requirement
- Multiple solutions being developed in community
- Options considered: Istio, linkerd, linkerd2 and connect - [FCP : Service Mesh](https://docs.google.com/document/u/0/d/1gCm2v-yQSViVN4wTrYnztCroKbWm6CQsm87HBAEjA8I/edit)
- Why istio?
    - It is most feature rich.
    - Has a very large active developer community.
    - First class support for Kubernetes.
    - Highly configurable.
    - Istio resources can be easily managed via declarative configs.
    - Backed by Google, IBM, Lyft, etc;

## Architecture

![Untitled](Istio/Untitled%201.png)

Istio has two main layers

- **Control Plane:** Control plane watches services and pods, CRDs in the kubernetes cluster(s) and then programs the data plane(envoy) configuration
- **Data Plane:** Uses an envoy proxy which runs as a sidecar in the application pod. It intercepts both incoming and outgoing tcp traffic via iptables rules. Once intercepted, traffic is intelligently routed by the envoy to the correct endpoint.

Below are main components of Istio Control Plane

- Istiod
    - Pilot - *Service Discovery, Config propagation*
    - Citadel - *Provides keys & certificates*
    - Galley - *Configuration validation & translation. The Istio CRDs created, and the translation of K8s CRDs into internal Istio objects is done by Galley.*
- Istio CNI - *Configure iptables rules to intercept traffic*

<aside>
💡 Does this remoove the use of ELBs altogether? All communication is proxy to proxy now? In the Byteshare they said we have reduced one network hop, since now the only network hop is from proxy to proxy. Whaaaat? Then is load balancing done by the Envoy proxy?

Yes to all of these.

</aside>

You can also configure proxy ← → prroxy communication to be in HTTP 2.0, while your original services remain in 1.0

Below are main components of Istio Data Plane

- Istio Proxy
    - Deployed as sidecar running along side app container
    - Transparently intercepts ingress/egress traffic
    - Uses an extended version of the Envoy proxy
    - Core part of functionality offered by Istio

## Architecture: What makes sidecar possible in K8s?

How can Istio use a sidecar without worrying about that extra network hop?

![Untitled](Istio/Untitled%202.png)

Diff uts namespace allows for having separate host names for containers wtihin the same pod.

Diff mount (mny) namespace allows for containers to treat that file system is exclusively for them.

But the pid namespace for containers withing a pod, the userids, ips, and the network namespace for all containers in a pod is shared. 

One container running one process can see the pid of another container.

Inter-process communication (ipc) is also shared.

The network namespace, ip table rules, sockets, all these are common for all the containers. This is why we can treat the entire pod as a single network namespace and communication for any application container can be tracked by using ip table rules.

This makes a sidecar possible, where you can intercept the traffic within the same network namespace without introducing an extra network hop.

## What is Envoy?

This is a network proxy that is designed to work alongside your app on your an application container. This is deployed in the same pod and intercepts all ingress and egress traffic for you application.

Nginx and other popular proxies are typically reverse proxies; where proxy is deloyed someplace else, but then you have config to route requests to your backend applcation pods.

- Aims to make network transparent to application
- Designed to run alongside app container as sidecar
- It can also do L3/L4 proxy for TCP/UDP
- Support HTTP L7 routing
    - Path
    - User/Authority
    - Content type, e.g. cookies
    - Http Headers
- Supports (can understand communication happening in these protocols and work on those requests)
    
    
    | TCP | UDP* | HTTP | HTTP2 | GRPC |
    | --- | --- | --- | --- | --- |
    
    ![Untitled](Istio/Untitled%203.png)
    

Just to be clear, sidecars will definitely consume more resources than dedicated elbs; multiple different sidecars consume more resources than a single dedicated elb. The reason we’re making this shift is because of the functionalities that are offered by this pattern.

## Benefits / Use Cases for Istio

Use-cases which Istio solves are listed as following :

(This was specific to Flipkart, noted in the doc)

## Envoy Performance Testing

- The Envoy proxy uses 0.35 vCPU and 40 MB memory per 1000 requests per second going through the proxy.
- The Envoy proxy adds 2.65 ms to the 90th percentile latency.

These resources will be an overhead in our pods

## Pod Creating and Auto Injection With Istio

![Untitled](Istio/Untitled%204.png)

A mutating webhook will inject the istio sidecar as an additional container in you pod spec, so the manifest that goes to the k8s apiserver will have this sidecar as well. On the node, kubelet will then create this pod with both these containers (the app and the sidecar).

This injection will be done based on your namespace label (simlar to how MTL sidecars are injected at the moment).

Resource usage by the proxy can be overriden by annotations, but these should typically not be changed and the default values should be used.

Certain ports that should not be used can also be configured via annotations.

Istio CNI configures the ip table rules that ensures that traffic sent from and to application is goes via proxy.

<aside>
💡 Why do we intercept the ingress traffic?
Envoy can also do things like mutual TLS. The app isn’t aware of any certificates and things. You can also implement authorization and athentication using Envoy. Such use cases will require interception of ingress traffic.

</aside>

## How does packet flow from sidecar to app?

![Untitled](Istio/Untitled%205.png)

There is a new connection between sidecar and app. The app will see the source IP as [localhost](http://localhost) only. You’ll have to use xforward or something to identify original source IPs. Envoy also has transparent proxy option but that isn’t enabled in FCP at the moment.

For outbound traffic, your packet will be intercepted through ip table rules, and it’ll be sent to the port listened to by sidecar.

That’s where the HTTP request parsing happens, and any path based routing, any traffic routing config is executed and the packet that’s coming out will have the actual endpoint of the pod it needs to hit, and this packet will go directly out of the pod.