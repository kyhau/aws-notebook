# Elastic Load Balancing (ELB)

- Elastic Load Balancing automatically distributes incoming application traffic across multiple targets, 
  such as EC2 instances, containers, IP addresses, and Lambda functions. 
- It can handle the varying load of your application traffic in a single AZ or across multiple AZs. 

It offers **3** types of load balancers that all feature the high availability, automatic scaling, and robust
  security necessary to make your applications **fault tolerant**.

1. [Application Load Balancer](#application-load-balancer-alb)
2. [Network Load Balancer](#network-load-balancer-nlb)
3. [Classic Load Balancer](#classic-load-balancer-clb)

See also [Load Balancer Comparisons](
https://aws.amazon.com/elasticloadbalancing/features/#Details_for_Elastic_Load_Balancing_Products).

More about NLB:
- [NLB vs. CLB Timeout](#nlb-vs-clb-timeout)
- [Connections time out for requests from a target to its load balancer](#nlb---connections-time-out-for-requests-from-a-target-targettypeinstance-to-its-load-balancer)
- [NLB Access Log Limitation](#nlb-access-log-limitation)


---

## Application Load Balancer (ALB)

- It is best suited for load balancing of HTTP and HTTPS traffic
- It provides advanced request routing targeted at the delivery of modern application architectures, including
  microservices and containers. 
- Operating at the **individual request level (Layer 7)**, it routes traffic to targets within Amazon VPC based on the
  content of the request.

## Network Load Balancer (NLB)

- It is best suited for load balancing of Transmission Control Protocol (TCP), User Datagram Protocol (UDP) and
  Transport Layer Security (TLS) traffic where extreme performance is required. 
- Operating at the **connection level (Layer 4)**, it routes traffic to targets within Amazon VPC and is capable of
  handling millions of requests per second while maintaining ultra-low latencies. 
- It is also optimized to handle sudden and volatile traffic patterns.
- You can now host multiple secure applications, each with its own TLS certificate, on a single load balancer listener. 
  This allows SaaS applications and hosting services to run behind the same load balancer, improving your service
  security posture, and simplifying management and operations. 
([whats-new/2019/09](https://aws.amazon.com/about-aws/whats-new/2019/09/elastic-load-balancing-network-load-balancers-now-supports-multiple-tls-certificates-using-server-name-indication/))

## Classic Load Balancer (CLB)

- It provides basic load balancing across multiple EC2 instances and operates at both the **request level (Layer 7) and
  connection level (Layer 4)**.
- It is intended for applications that were built within the EC2-Classic network.


# NLB vs. CLB Timeout

Ref: https://medium.com/tenable-techblog/lessons-from-aws-nlb-timeouts-5028a8f65dda

> For each request that a client makes through a Network Load Balancer, the state of that connection is tracked. 
> The connection is terminated by the target. If no data is sent through the connection by either the client or target
> for longer than the idle timeout, the connection is closed. If a client sends data after the idle timeout period
> elapses, it receives a TCP RST packet to indicate that the connection is no longer valid.

In other words, AWS NLBs silently terminates your connection upon idle timeout. If an application tries to send data
on the socket after idle timeout, it receives an RST packet.

- An ELB’s idle timeout setting is adjustable, and defaults to 60 seconds. 
  When connections expire through idle timeout, ELBs sends a FIN packet to each connected party. 
  This translates to a socket closed event in the application layer.

- NLBs have an idle timeout of 350 seconds which cannot be changed. 
  When connections expire through idle timeout, NLBs terminate the connections silently. 
  An application that is not aware of this timeout would attempt to send data to the same socket. At that point,
  the NLB would notify the application that the connection has been terminated by sending it an RST packet.

# NLB - Connections time out for requests from a target (TargetType=Instance) to its load balancer

[Source](
https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-troubleshooting.html#loopback-timeout):

- Check whether you have an internal load balancer with targets registered by instance ID. Internal load balancers
  do not support hairpinning or loopback. When you register targets by **instance ID**, the source IP addresses of
  clients are preserved. 
- If an instance is a client of an internal load balancer that it is registered with by **instance ID**, the
  connection succeeds only if the request is routed to a different instance. Otherwise, the source and destination IP
  addresses are the same and the connection times out. 

# NLB Access Log Limitation

**Issue:** Enabled access log for NLB but no log were generated in the S3 bucket.

From AWS Support: 
Please note that NLB access logs are created only if the load balancer has a TLS listener and they contain information 
only about TLS requests [1].  NLB access log is not currently support on TCP, UDP or TCP_UDP listener.

Although the fields are slightly different, but using VPC logs [2][3] to record the flow messages for each NLB network
interface is an alternative that you can consider. To find the network interfaces of your NLB, please type the name of
your NLB in the search field of Network Interfaces navigation pane [4].

In addition, you can also consider using the TLS listener [5] for your NLB if it meets your needs.

References:
- [1] Access Logs for Your Network Load Balancer - https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-access-logs.html 
- [2] VPC Flow Logs - https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html 
- [3] Monitor Your Network Load Balancers - https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-monitoring.html 
- [4] https://console.aws.amazon.com/ec2/ 
- [5] TLS Listeners for Your Network Load Balancer - https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-tls-listener.html 
