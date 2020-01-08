# Networking

Topics
- [Network Bandwidth for EC2 Instances](#network-bandwidth-for-ec2-instances)

---
## Network Bandwidth for EC2 Instances

Actual bandwidth is dependent on the instance type and size ([Ref](
https://aws.amazon.com/blogs/aws/the-floodgates-are-open-increased-network-bandwidth-for-ec2-instances/)).
1. EC2 to S3: 25 Gbps
2. EC2 to EC2 (in the same or different AZs within a region):
   - 5 Gbps of bandwidth for single-flow traffic, or
   - 25 Gbps of bandwidth for multi-flow traffic 
3. EC2 to EC2 (Cluster Placement Group): 
   - 10 Gbps of lower-latency bandwidth for single-flow traffic, or
   - 25 Gbps of lower-latency bandwidth for multi-flow traffic

**A flow represents a single, point-to-point network connection) by using private IPv4 or IPv6 addresses.

Enhanced Networking Types 
([Ref-Linux](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/enhanced-networking.html), 
[Ref-Windows](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/enhanced-networking.html))
1. ENA (Elastic Network Adapters)
   - Support network speeds of up to 100 Gbps
   - ENA Support needs to be flagged when registering an AMI.
2. Intel 82599 Virtual Function (VF) interface
   - Support network speeds of up to 10 Gbps
