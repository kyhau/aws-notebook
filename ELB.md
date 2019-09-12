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
- You can now host multiple secure applications, each with its own TLS certificate, on a single load balancer listener. This allows SaaS applications and hosting services to run behind the same load balancer, improving your service security posture, and simplifying management and operations. 
([whats-new/2019/09](https://aws.amazon.com/about-aws/whats-new/2019/09/elastic-load-balancing-network-load-balancers-now-supports-multiple-tls-certificates-using-server-name-indication/))

## Classic Load Balancer (CLB)

- It provides basic load balancing across multiple EC2 instances and operates at both the **request level (Layer 7) and
  connection level (Layer 4)**.
- It is intended for applications that were built within the EC2-Classic network.
