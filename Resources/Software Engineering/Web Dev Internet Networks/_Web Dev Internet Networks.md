# Web Dev / Internet / Networks

# Web Dev Starter

[https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/How_the_Web_works](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/How_the_Web_works)

## Stateful vs Stateless

In this context and very simply put: stateless means that every request contains all the information needed to fulfull that request. Http is a stateless protocol: all information is sent by the user with every request.

For example pop3 (for email) is not stateless: first a user logs in with username and password, and from then on types requests. This means the pop3 server must *remember* sessions, because this login information is not sent with every following request.

Another way to think about it is in terms of variables. With stateless http requests, after a request comes through all your variables go away (stateless = no state) . With at stateful environment like a desktop app those variables can remain in memory and accessible (stateful).

# REST vs SOAP

| **SOAP API** | **REST API** |
| --- | --- |
| Relies on SOAP (Simple Object Access Protocol) | Relies on REST (Representational State Transfer) architecture using HTTP. |
| Transports data in standard XML format. | Generally transports data in JSON. It is based on URI. Because REST follows stateless model, REST does not enforces message format as XML or JSON etc. |
| Because it is XML based and relies on SOAP, it works with WSDL | It works with GET, POST, PUT, DELETE |
| Works over HTTP, HTTPS, SMTP, XMPP | Works over HTTP and HTTPS |
| Highly structured/typed | Less structured -> less bulky data |
| Designed with large enterprise applications in mind | Designed with mobile devices in mind |

While SOAP was once widely used, REST has become the more popular choice for building web services in recent years. This is due to several factors, including its simplicity, scalability, and support for multiple data formats. REST has also gained popularity due to the growth of mobile and IoT devices, which require lightweight communication protocols.

However, SOAP is still used in some enterprise-level applications, especially those that require more advanced security features or support for legacy systems. SOAP is also still used in some industries, such as finance and healthcare, where there is a greater emphasis on data integrity and reliability

# IP Addresses

Your IP address probably looks like

```java
192.168.1.x
```

This also ships with two other pieces of information

```java
IP: 192.168.1.x
Netmask: 255.255.255.0 (11111111.11111111.11111111.00000000)
Gateway: 192.168.1.x
```

Out of these things, if first 3 octets of netmask are 255, the first three octets of all the devices in your network will remain the same.

The last digit can change, and can be one of 253 values. Why not 256? Because three values are reserved.

- The Router: 192.168.1.0
- The Broadcast Address: 192.168.1.255
- The Gateway: Can be any one of the other 254 values.

Here, the gateway is the one powering DHCP (which gives all the devices on your network their dynamic IPs) and DNS (which resolves domain names to actual IP values).

So 192.168.1. is a **Network**, which can have up to 253 **Hosts**

These are recurrent terms in subnetting; network, and hosts per network.

## Classing IPs

| **Class** | **Address Range** | **Subnet masking** | **Example IP** | **Leading bits** | **Max number of networks** | Max hosts per network | Network range possible | **Application** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| IP Class A | 1 to 126 | 255.0.0.0 | 1.1.1.1 | 8 | 128 | 16,777,214 | 1. to 127. | Used for large number of hosts. |
| IP Class B | 128 to 191 | 255.255.0.0 | 128.1.1.1 | 16 | 16384 | 65,534 | 128.0. to 191.255. | Used for medium size network. |
| IP Class C | 192 to 223 | 255.255.255.0 | 192.1.11. | 24 | 2097157 |  |  | Used for local area network. |
| IP Class D | 224 to 239 | NA | NA | NA | NA |  |  | Reserve for multi-tasking;  |
| IP Class E | 240 to 254 | NA | NA | NA | NA |  |  | This class is reserved for research and Development Purposes. |

```java
Class A: 1 -> 0000 0001
	// 127 is loopback
Class B: 128 -> 1000 0000 
Class C: 192 -> 1100 0000
Class D: 224 -> 1110 0000
Class E: 240 -> 1111 0000
```

## Real World Tidbits

- Classful is a dead term practically. Most enterprises use the 10.0.0.0-10.255.255.255 private range (since it apparently allows the most amount of usable addresses), subnetting it further into /24 blocks for users at HOs and branches, or for servers in their DCs, /29 or /30 for connections. Things are NAT/PAT'ed at the edge firewalls or edge routers, for in-out access and out-in access.
Lots of great points here ([https://www.reddit.com/r/ccna/comments/cr57wz/ip_classes_usage_class_a_vs_class_b/](https://www.reddit.com/r/ccna/comments/cr57wz/ip_classes_usage_class_a_vs_class_b/))
- They can also implement different private ranges (172, 192 or just other unused 10) situationally (e.g. some branches, business partners, etc.)
- To the network engineer 10. ; 172.16-31. ; and 192.168. ; are all known and easily identified as internal IPs.
- 

## Loopback Addresses

```java
ping 127.0.0.1 
```

Will let you check if your network interfaces in your laptop are up and running.

For some reason, there are 16 Million of these; all addresses from 127.0.0.0 to 127.255.255.255.

## Running Out Of IPs

There are only about 2^{32} 4.3 Billion possible IP addresses considering all these numbers.

## Public vs Private IP

Introduced with RFC1918

| **Public IP address** | **Private IP address** |
| --- | --- |
| External (global) reach | Internal (local) reach |
| Used for communicating outside your private network, over the internet | Used for communicating within your private network, with other devices in your home or office |
| A unique numeric code never reused by other devices | A non-unique numeric code that may be reused by other devices in other private networks |
| Found by Googling: "What is my IP address?" | Found via your device’s internal settings |
| Assigned and controlled by your internet service provider | Assigned to your specific device within a private network |
| Not free | Free |
| Any number not included in the reserved private IP address range
Example: 8.8.8.8. | 10.0.0.0 — 10.255.255.255; FROM CLASS A
172.16.0.0 — 172.31.255.255; FROM CLASS B
192.168.0.0 — 192.168.255.255 FROM CLASS C
Were taken out, and reserved as private IP addresses.
They retained the subnet masks from their original classes.
Example: 10.11.12.13 |

## Network Address Translation (NAT)

It would be impossible to assign every device in your home network a public IP address.

So instead, your router gives out private IP addresses to these devices.

Your router is assigned a public IP by your ISP.

When your device connect to any other public IP, your router will convert it’s private IP to your public IP using NAT. When the public IP server responds, the router will know which device to forward this response to.

So every device on your network, has a common identity.

## IPv6

Even after RFC1918 and NAT, we were short of IP addresses. So we came up with IPv6. In certain places, we can only use public IPs; say your cellphone is connected to it’s cellular network, it only has only one IP, and that’s an IPv6 public IP. But most websites still use IPv4 addresses.

## Static IP vs Dynamic IP

When a device is assigned a static IP address, the address does not change. Most devices use dynamic IP addresses, which are assigned by the network when they connect and change over time.

# 7 Layer Model ←→ TCP/IP Model

![Untitled](Web%20Dev%20Internet%20Networks/Untitled.png)

## L4 vs L7 Load Balancer

More details on [https://levelup.gitconnected.com/l4-vs-l7-load-balancing-d2012e271f56](https://levelup.gitconnected.com/l4-vs-l7-load-balancing-d2012e271f56)

### L4 Load Balancer

**L4 load balancing** offers traffic management of transactions at the network protocol layer (TCP/UDP). L4 load balancing delivers traffic with limited network information with a load balancing algorithm (i.e. round-robin) and by calculating the best server based on fewest connections and fastest server response times.

- Differences
    - Done at L3 → L4 layers
    - We are unaware of data in requests, we only know IPs and ports.
    - When client makes a request to L4 LB, it uses the same connection that the client made with the LB, to connect to one of the upstream servers.
- Cons
    - In keep-alive connections, requests keep getting sent to the originally selected client since we reuse the same TCP connection. Load isn’t balanced in that case.
    - Doesn’t offer authentication.
    - No TLS termination.

### L7 Load Balancer

**L7 load balancing** works at the highest level of the OSI model. L7 bases its routing decisions on various characteristics of the HTTP/HTTPS header, the content of the message, the URL type, and information in cookies.

- Differences
    - Done at L5 → L7 layers
    - We are aware of the data in the requests.
    - When a client makes a request, it creates a TCP connection with the LB. The LB then creates a new TCP connection with one of the upstream servers.
- Pros
    - In the case of multiplexed/keep-alive protocols, L7 load balancer creates a TCP connection with every upstream for a single client connection rather than choosing a single upstream. This means that requests are distributed over various upstreams as expected.
    - Offers Authentication, if headers are missing, the server can send 401 responses.
    - The load balancer can also act as a TLS termination proxy and decrypt SSL encrypted traffic for the backend service, reducing load on service for this CPU intensive task.

# Same Origin Policy

The **same-origin policy** is a critical security mechanism that restricts how a document or script loaded by one [origin](https://developer.mozilla.org/en-US/docs/Glossary/Origin) can interact with a resource from another origin.

It helps isolate potentially malicious documents, reducing possible attack vectors.

For eg. it prevents a malicious website on the Internet from running JS in a browser to read data from a third-party webmail service (which the user is signed into) or a company intranet (which is protected from direct access by the attacker by not having a public IP address) and relaying that data to the attacker.

[](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAMAAAAoLQ9TAAAAaVBMVEUVFBoAAAsAAAMMCxMQDxZ1dXiJiYtNTE8AAAA5OTz////KysuPjpArKi/c3N1paWsEAA3l5eaysrOamZvU1NRmZmi/vsC4uLkzMzcHBQ9ZWVyAgILe3t/Ozc6VlZb49/ghICZDQkbs7OzV+rXNAAAAdklEQVR4AY1OtQHAIBDEubi7Z/8d44GW7/6cOB5lnBDGxP8zqTSHUt6PwA9CRH6cpJ9BZnmIIiirD0BdNCEyGRvAb+sQPn6g633UahgNgLxBPUyzBZbqAvz1B/jmA/W0A1nyAlrSrhr6jspvGKdXMOuuPYK43AlA5QbKve4JeQAAAABJRU5ErkJggg==)

**The SOP defines the following rules:**

- Each site has its own resources like DOM, cookies, and Javascript namespace
- Each page takes its origin from its URL, which is usually the schema, domain, and port
- Scripts run in the context of the origin they are loaded from
- Many resources, like media and images, are passive and do not have access to objects and resources in the context they were loaded

An origin is defined as a combination of URI scheme, host name, and port number. For example, the URL https://example-website.com:443/example/path has the origin https://example-website.com:443, where the scheme is "https", the domain is "example-website.com", and the port is "443"

# DNS

Converts host name into actual IP where server is hosted

![Untitled](Web%20Dev%20Internet%20Networks/Untitled%201.png)

1. Browser makes DNS query to figure out IP of your website
2. First it checks in Browser local cache
3. If not found, this lookup is made with DNS Recursive Resolver