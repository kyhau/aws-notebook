# Host-based Security

Table of Contents

- [AWS Host / Hypervisor Security (disk/memory)](#aws-host--hypervisor-security-diskmemory)
- [Host Proxy Servers  (aka. Instance Proxy Servers)](#host-proxy-servers--aka-instance-proxy-servers)
- [Host-based IDS/IPS (Intrusion Detection/Prevention System)](#host-based-idsips-intrusion-detectionprevention-system)
- [SSM (Systems Manager)](#systems-manager-ssm)
- [Packet Capture on EC2](#packet-capture-on-ec2)


## AWS Host / Hypervisor Security (disk/memory)

Prevent data leakage between clients within their shared environment.

1. Isolation
   1. Instances are always isolated from other customer instances.
   2. Instances have **no direct access to hardware**, and must go via the **hypervisor and firwall system**.
   3. Use **Dedicated Host** instead if you need the instances run on a physical server that is dedicated for your uses.
2. Memory
   1. Host memory is allocated to an instance **scrubbed** (i.e. zero'd out).
   2. When un-allocating memory, the **hyervisor** zeros it out *before* returning it to the pool.
3. Disk
   1. EBS volumes are provided to instance in a **zero'd** state; i.e. this zeroing occurs *immediately before use*.
   2. If you have specific deletion requirements, you need to do this *before terminating* the instance/deleting the
      volume.


## Host Proxy Servers  (aka. Instance Proxy Servers)

1. Filtering within AWS is performed at 2 points:
   1. **Security Groups** attached to network interfaces, and
   2. **Network ACLs (NACLs)** attached to subnets within VPCs.
2. Security Groups and NACLs have viability of protocols, IPs, CIDRs and ports. 
   1. They cannot filter DNS names.
   2. They cannot decide between allowing and denying traffic based on any form of authentication.
3. If authentication or additional intelligence beyond IP/CIDR/port/protocol is needed, a proxy server or an
   **enhanced NAT architecture** is required.
4. The **proxy server** can be much more granular in allowing or denying traffic; either using
   1. Authentication (e.g. username/password, ID federation), or
   2. Higher level traffic analysis: **DNS name, web path**, etc.

```
 ________________________________________________________________________________ 
 
 Public Subnet               _______
                      SG1   |  NAT  |     SG1 allows incoming traffic from any
                            |_______|     instances SG2 is attached to the proxy.
                                ^
 _______________________________|________________________________________________ 
                                |
 Services Subnet             ___v___
                      SG2   | proxy |     SG2 only allows traffic from SG3
                            |_______|     attached instances.
                                ^
 _______________________________|________________________________________________ 
                                |
 Private Subnet             ____|_____
                      SG3  | Instance |   The proxy server can be much more
                           |__________|   granular in allowing or denying traffic.
 ________________________________________________________________________________ 
```

## Host-based IDS/IPS (Intrusion Detection/Prevention System)

1. The Intrusion Detection System (IDS), is a category of software designed purely to monitoring resources and identify
   any suspicious activity that could indicate a current, past, or future threat.
2. A host-based IDS solution compliments the features available within AWS:
   1. **WAF (Web Application Firewall)** provides edge security before a threat arrives at your environment edge.
   2. **IDS Appliance** monitors and analyzes data as it moves into your platform.
   3. **AWS Config** ensures a stable and compliant configuration of account level aspects.
   4. **SSM (Systems Manager)** ensures compute resources are compliant with patch levels.
   5. **Inspector** reviews resources for known exploits and questionable OS/Software configurations.
   6. **Hosted-based IDS/IPS** solutions offer key features to help protect your EC2 instances. This includes alerting
      administrators of malicious activity and policy validations, as well as identifying and taking action against
      attacks. From Marketplace, e.g. McAfee Virtual Network Security Platform, AlertLogic Threat Manager, etc.


## Systems Manager (SSM)

## Packet Capture on EC2

