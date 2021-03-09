# Networking - key notes

- [Uncategorised - IP, CIDR, ENI, NAT Gateway, VPC sharing](#uncategorised---ip-cidr-eni-nat-gateway-vpc-sharing)
- [VPC Flow Logs](#vpc-flog-logs)
- [Networking scenarios](#networking-scenarios)
- [BGP](#bgp)
- [VPN](#vpn)
- [DX](#dx)
- [Routing and redundant connections](#routing-and-redundant-connections)
- [Transit Gateway](#transit-gateway)
- [VPC Endpoints](#vpc-endpoints)
- [Gateway Load Balancer](#gateway-load-balancer)
- [Enhanced Networking (network speeds)](#enhanced-networking-network-speeds)
- [Placement groups](#placement-groups)
- [DNS, R53](#dns-r53)
- [AD](#ad)
- [Resolving DNS Queries Between VPCs and Your Network](#resolving-dns-queries-between-vpcs-and-your-network)
- [ELB](#elb)
- [CloudHSM](#cloudhsm)
- [CloudFront, WAF](#cloudfront-waf)
- [Workspaces](#workspaces)
- [AppStream 2.0](#appstream-20)

---

### Uncategorised - IP, CIDR, ENI, NAT Gateway, VPC sharing

1. IPv4: largest, **/16**  (Host bit mask `11111111 11111111 hhhhhhhh hhhhhhhh` → 2^16 = 65536 IPs)
1. IPV4: smallest, **/28**  (Host bit mask `11111111 11111111 11111111 1111hhhh` → 2^(32-28) = 2^4 = 16 IPs)
1. IPv6: VPC fixed size of **/56**
1. IPv6: Subnet fixed size of **/64**
1. Ephemeral range: **1024 - 65535**
1. **5** addresses reserved per subnet: `+0 network, +1 VPC router, +2 DNS, +3 future, .255 broadcast`
1. ELBs need a **/27** at a minimum (not /28) and have at least 8 free IP addresses. ([Source](https://docs.aws.amazon.com/elasticloadbalancing/latest/classic/elb-backend-instances.html))
1. CIDR block from private (non-publicly routable) IP addresses can be assigned.
   ```
   - 10.0.0.0/8         10.0.0.0 – 10.255.255.255
   - 172.16.0.0/12      172.16.0.0 – 172.31.255.255
   - 192.168.0.0/16     192.168.0.0 – 192.168.255.255
   ```
1. Secondary IPv4 CIDR blocks
   - Can use: non-RFC 1918, or `100.64.0.0/10` (RFC 6598)
   - Cannot use: `198.19.0.0/16`
   - Primary `10.0.0.0/8`  =>  any other CIDR from `10.0.0.0/8` range
      - If primary CIDR falls within `10.0.0.0/15`, cannot add a CIDR block from `10.0.0.0/16` 
   - Primary `172.16.0.0/12`  =>  any other CIDR from `172.16.0.0/12`, except `172.31.0.0/16`
   - Primary `192.168.0.0/16` =>  any other CIDR from 192.168.0.0/16
   - Primary `198.19.0.0/16`
   - Primary Non-RFC 1918, `100.64.0.0/10`
1. Calculate IP range
   ```
   E.g. 10.0.8.0/21 encompasses addresses from 10.0.8.0 to 10.0.15.255. 
   Calculation: (Source)
   10.0.8.0 in binary:    00000010 00000000 00001000 00000000
   Network mask (21):     11111111 11111111 11111000 00000000 (twenty-one 1s)
                           ----------------------------------- [Logical AND]
   FirstIP/NetworkAddr:   00000010 00000000 00001000 00000000 ----> 10.0.8.0
   
   10.0.8.0 in binary:    00000010 00000000 00001000 00000000
   Host bit mask (21):    00000000 00000000 00000hhh hhhhhhhh ----> 2^11 = 2048 IPs
                           ----------------------------------- [Force host bits]
   LastIP/BroadcastAddr:  00000010 00000000 00001111 11111111 ----> 10.0.15.255
   ```
1. `curl http://169.254.169.254/latest/meta-data/public-ipv4`
1. Broadcasting is not allowed by AWS.
1. Max. **5 VPCs** (and **5 IGWs**) per Region by default (can have 100+ of VPCs per Region).
1. NACL needs an outbound rule allowing return traffic on ports (e.g. 32768 - 65535).
1. In case of a DDoS, the fastest way to stop incoming requests is to delete the default NACL in a VPC.
1. A **subnet** can have **one NACL**.  **Max 10 subnets** can use **one NACL**. 
1. A **subnet** can have **one route table**. A **Route table** can be associated with **multiple subnets**.
1. Configuration elements scoped to an **ENI**: **SRC and DST check**, **SGs**, **Private IPs**, **Mac address**.
1. You can move an ENI between subnets, but not AZs.
1. Cannot detach primary (eth0) ENI interface.
1. **SRC and DST check** is an **ENI** setting. Disable this check for NAT instance or proxy server on EC2.
1. NAT Gateway: need EIP, AZ specific, **45 Gbps**, cannot ssh, cannot port forward
1. NAT Gateway: `ErrorPortAllocation > 0`  too many concurrent (55,000) connections, split the resources between multiple subnets and create multiple NAT gateways per AZ.
1. VPC peering relationship does not extend to VPN, DX, IGW, VPC endpoint.
1. If require more than **125 peering connections per VPC**, consider using Direct Connect.
1. **VPC sharing** uses **RAM** to share subnets across accounts within the same AWS organization.

### VPC Flog Logs
1. VPC Flow Logs can attach to **VPC**, **Subnet**, and **ENI**.
2. Both VPC Flow Logs and GuardDuty (that uses VPC Flow Logs) can **detect port scanning**.
3. `<version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> <protocol> <packets> <bytes> <start> <end> <action> <log-status>` <br>
   `2 123456789010 eni-xxxx 203.0.113.12 172.31.16.139 0 0 1 4 336 1432917027 1432917142 ACCEPT OK`
4. VPC Flow Log - protocols: 1/ICMP, 6/TCP, 17/UDP, 27/RDP
5. [VPC Flow Log examples](VpcFlowLogs.md)
6. VPC Flow Logs do NOT deliver real-time data.

### Networking scenarios
1. Internet
   ```
   1. IGW (public IPs):   main route table:             0.0.0.0/0, igw-id
   2. NAT gateway + IGW:  private subnet route table:   0.0.0.0/0, nat-gw-id 
                          public subnet route table:    0.0.0.0/0, igw-id 
   3. Egress-Only IGW:    private subnet route table:   ::/0,      e-igw-id
   ```
2. VPN
   1. VPN: **`IGW (EC2-based VPN, EIP) ← vpn → IGW (EC2-based VPN, EIP)`**
   2. VPN: **`VGW (VPC) ← vpn → IGW (EC2-based VPN, VPC)`**
3. VGW
   1. VPN: **`VGW (VPC-1) ← site-to-site vpn → CGW ← site-to-site vpn →  VGW (VPC-2)`**
      - supports transitive routing between VPCs
   2. DX: **`VGW (VPC) ← private VIF → CGW`**
      - supports transitive routing between VPCs
   3. DX: **`VGW (VPC) ← vgw association → DX Gateway ← private VIF → CGW`**
      - Private VIF and DX gateway must be in the same AWS account to use DX gateway functionality.
      - VGW(s) can be in different AWS accounts then the account owning the DX gateway.
4. Transit Gateway
   1. VPN: **`VPC ← vpc attachment → Transit Gateway ← site-to-site vpn, vpn attachment → CGW`**
   2. DX: **`VPC ← vpc attachment → Transit Gateway ← transit VIF → CGW`**
   3. DX: **`VPC ← vpc attachment → Transit Gateway ← tg association → DX Gateway ← transit VIF → CGW`**
      - Transit VIF and DX gateway must be in the same AWS account to use DX gateway functionality.
      - Transit Gateway(s) can be in different AWS accounts then the account owning the DX gateway.
5. Transit VPC
   1. Transit VPC (a) - Transit VPC to Spoke VPCs
      - **`IGW (EC2-based VPN, transit VPC) ← vpn → VGW (spoke VPC)`**
      - **`IGW (EC2-based VPN, transit VPC) ← vpn → (EC2-based VPN, spoke VPC)`**
   1. Transit VPC (b) - Transit VPC to Network
      - **`IGW (EC2-based VPN, transit VPC) ← vpn → detached VGW ← private VIF → CGW`**
      - **`IGW (EC2-based VPN, transit VPC) ← vpn → CGW`**
   1. **Detached VGWs** can advertise BGP prefixes learned from multiple endpoints, unlike a VGW attached a VPC that only advertises CIDRs.
   1. **Transit VPC supports multiple accounts and regions.** DX Gateway supports multi account since 2019-03, but it is limited to 10 VGWs, 3 Transit gateways, 30 (private/transit) VIFs.
   1. **VPN appliances should be deployed into separate AZs and enable dynamic routing for maximum availability.**
6. VPN CloudHub
   1. **`VGW (VPC) ← site-to-site vpns (diff ASN) → CGWs (offices)`**
   2. Cannot use a DX gateway with CloudHub. Can attach a DX VIF directly to a VGW to support CloudHub.
7. AWS Client VPN
   1. **`AWS Client VPN Endpoint (Subnet) ← vpn → Client device`** (vpn client e.g. openvpn client)

### BGP
1. **Max 100 BGP advertised routes** per route table. If > 100, **use default route**, or **summarize routes**.
1. BGP session is going from established to idle state (means routes > 100)
1. **Private VIFs**: up to **100 prefixes** advertised over BGP
1. **Public VIFs**: up to **1000** prefixes advertised over BGP 
1. **BGP: TCP/179**; make sure not being blocked by firewall or ACL rules

### VPN
1. AWS Site-to-Site VPN can attach to **VGW** or **Transit Gateway**.
1. VPN connection: **UDP/500 (IKE), UDP/4500 (NAT-Traversal), and IP protocol 50 (ESP)**.
1. Each connection provides **two redundant IPsec tunnels**.
1. Each VGW gives **2 public IP addresses as endpoints** in **2 AZs**.
1. How to avoid [asymmetric routing](https://edge-cloud.github.io/2019/08/16/aws-dxgw-with-ipsec-vpn-backup/#desired-architecture)?
   - Prepend an AS_PATH (shortest preferred)
   - Set the multi-exit discriminator (MED) metric attribute (lowest better)
1. A VPN connection may lose if being idle for **10s x 3**. Use a keep-alive tool to send continuous traffic.
1. IPsec VPN (VGW) throughput up to **1.25 Gbps**.
1. EC2-based VPN throughput can reach **2.0 - 2.5 Gbps**.
1. **10 IPsec connections per VGW/VPC**.
1. **50 CGW per AWS account per AWS Region** (i.e. cannot connect to more than 50 sites)
1. Not recommended using AWS Managed VPN as a backup for DX connections with speeds > 1 Gbps.
1. Site-to-Site VPN cannot access NLB, EFS, VPC endpoints, PrivateLink endpoints, VPC DNS IP, AWS public IP range.
1. VGW doesn’t initiate IPsec negotiation. CGW must initiate the tunnels.
1. Not support: IPv6 traffic, Path MTU Discovery, AES 128-bit encryption, multicast.

### DX
1. Single-mode optical fiber, 802.1 Q, BGP, BGP MD5, manually set port speed and full-duplex mode (disable auto-negotiation for port)
1. Each **Dedicated Connection**: up to **50 public/private VIFs, 1 transit VIF**.
1. Each **Host Connection**: **1 public/private/transit VIF**.
1. Each **LAG**: Max of **four connections, same bandwidth, terminate at same DX endpoint. Max 50 VIFs. 1 Gbps or 10 Gbps**.
1. All connections in a LAG run in an Active/Active configuration.
1. Each DX gateway: **10 VGWs, 3 Transit gateways, 30 (private/transit) VIFs**
1. A VGW is associated with one VPC.
1. A VGW can be associated to one DX gateway.
1. You can provision a single connection to any DX location in the US and use it to access public AWS services in all US Regions and AWS GovCloud (US).
1. Jumbo Frames for DX
   - MTU of a **private VIF** can be either **1500 or 9001 bytes (jumbo frames)**.
   - MTU of a **transit VIF** can be either **1500 or 8500 bytes (jumbo frames)**.
1. CloudWatch metrics for DX (**10-Gbps**): **ConnectionLightLevelTx, ConnectionLightLevelRx**
1. Public VIF:  any AWS public services globally, all IPs within, even public IPs of EC2
1. A private VIF can attach to one VGW or DX gateway.
1. A private VIF can connect to multiple VPCs, as a DX gateway can be associated to up to 10 VGWs.
1. 10 VPCs per private VIF.
1. For a private VIF, AWS only advertises the **entire VPC CIDR** over the BGP neighbor.
1. Setup VIF: gateway type (VGW ID), VLAN ID, BGP ASN, MD5 key, prefixes to advertise (public VIF), jumbo frame setting (private VIF)
   - If you are using a public ASN you must own it.
   - If you are using a private ASN, it must be in the 64512 - 65535 range.
1. All the intermediate devices are configured for VLAN tagging with appropriate VLAN ID, and VLAN-tagged traffic is preserved in the DX endpoint. Some network providers might also use **Q-in-Q tagging/tunneling**, which can alter your tagged VLAN.

### Routing and redundant connections
1. Advertise more specific routes
1. **Highest LOCAL_PREF in** (sending traffic to Amazon)
1. **Shortest AS_PATH out** (sending traffic from Amazon)
1. AS_PATH prepending does NOT work if you use a **private ASN for a public VIF**.
1. DX advertises prefixes with a **minimum path length of 3**.
1. DX advertises all public prefixes with the well-known **NO_EXPORT BGP community**.
1. The **NO_EXPORT BGP** community tag is supported for public VIFs, private VIFs, transit VIFs.
1. **LOCAL_PREF** BGP community tags are evaluated before any **AS_PATH** attribute.
1. If not specified, the default **LOCAL_PREF** is based on the distance to the DX location.
1. How to set up Active/Passive DX connection?
   - Two routers to two DX connections to avoid a single point of device failure.
   - A private VIF on each of the DX routers that terminate to the same VPC.
   - Use LOCAL_PREF and AS_PATH.
1. How to set up VPN as backup for DX?
   - Use the same VGW for both DX and the VPN connection to the VPC.
   - Advertise the same prefix for DX and the VPN.
1. Enable BFD when configuring multiple DX connections, or a single DX + VPN backup.
1. Asynchronous BFD is automatically enabled for DX VIFs, but does not take effect until configure it on router.
1. BFD liveness detection **300ms**. Otherwise default BGPs waits for keep-alive failures of **90s x 3**.

### Transit Gateway
1. Transit Gateway supports Inter-region peering. ([Source](https://aws.amazon.com/about-aws/whats-new/2020/04/aws-transit-gateway-now-supports-inter-region-peering-in-11-additional-regions/))
1. Transit Gateway attachment types: **VPC** or **VPN**
1. When you create a VPN attachment, it will create a site-to-site VPN for you.
1. You can use **ECMP routing** to get higher VPN bandwidth by aggregating multiple VPN connections.
1. Each Transit Gateway: **5000 attachments, 50 peering attachments, 20 transit gateway route tables, 10000 static routes, 20 DX gateways, 2 subnets for a VPC**
1. Each attachment can associate to 1 route table. 
1. **50 Gbps** per VPC, DX gateway, peered transit gateway, 
1. **1.25 Gbps** per VPN
1. Support **multicast** between VPCs.
1. Routes propagated to/from on-prem networks: **BGP (dynamic)**
1. When you attach a VPC to a Transit Gateway or resize an attached VPC, the VPC CIDR will propagate into the Transit
  Gateway route table using internal APIs (not BGP).
1. Routes in the Transit Gateway route table will not be propagated to the VPC’s route table. 
  VPC owner need to create static route to send traffic to the Transit Gateway.

### VPC Endpoints
1. **DNS resolution** is required within the VPC.
1. Gateway VPC endpoint
   - **Region specific** (need to be in the same region); support IPv4 only.
   - Attach **Endpoint policy**; not Security Group.
   - Prefix list ID: **pl-xxxxxxx**; Prefix list name: **com.amazonaws.us-east-1.s3**
   - **Subnet** route table: **Destination: pl-id-for-s3, Target: vpce-id**
1. Interface VPC endpoint (powered by AWS PrivateLink)
   - **AZ specific**
   - Through **IGW** (not using Interface Endpoint): DNS **sns.us-east-1.amazonaws.com, public IP**
   - Using Interface Endpoint: DNS **vpce-<id>.sns.<region>.amazonaws.com, private IP**
   - Using Interface Endpoint with Private DNS Name: DNS **sns.us-east-1.amazonaws.com, private IP**
   - To use **private DNS names**, 
      - Enable **Private DNS Name **(when creating new Interface VPC Endpoint)
      - Set **VPC settings to true: enableDnsHostnames, enableDnsSupport**.
1. Gateway Load Balancer endpoint (powered by AWS PrivateLink)
   - A VPC endpoint that provides private connectivity between virtual appliances in the service provider VPC and application servers in the service consumer VPC.
   - Traffic to and from a Gateway Load Balancer endpoint is configured using route tables. 
   - Traffic flows from the service consumer VPC over the Gateway Load Balancer endpoint to the Gateway Load Balancer in the service provider VPC, and then returns to the service consumer VPC. 
   - You must create the Gateway Load Balancer endpoint and the application servers in different subnets. This enables you to configure the Gateway Load Balancer endpoint as the next hop in the route table for the application subnet.
1. Gateway VPC Endpoint for S3
   1. Attach an **Endpoint Policy** to the endpoint to limit its functionality.
   2. Add a **route** in the route tables for any subnets where the gateway will be used. 
      - Subnet route table:  **Destination: pl-id-for-s3, Target: vpce-id**
   3. Use **bucket policies** to control access to buckets from specific endpoints, or specific VPCs.
1. If a Lambda function needs to access both VPC resources and the public internet, the VPC needs to have a 
  **NAT gateway** in a **public subnet**.

### Gateway Load Balancer
1. Gateway Load Balancers enable you to deploy, scale, and manage virtual appliances, such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems.
1. L3 (network layer), listens for all IP packets across all ports and forwards traffic to the target group that's specified in the listener rule. 
1. Use GENEVE protocol on port 6081. It supports a MTU (maximum transmission unit) size of 8500 bytes.
1. It maintains stickiness of flows to a specific target appliance using 5-tuple (for TCP/UDP flows) or 3-tuple (for non-TCP/UDP flows). 

### Enhanced Networking (network speeds)
1. Higher packet-per-second (PPS) performance, lower inter-instance latencies, very low network jitter:
   - Select an instance with support for single root I/O virtualization.
   - Ensure that proper OS drivers are installed.
1. ENA: **100 Gbps**
1. Intel 82599 Virtual Function (VF) interface: **10 Gbps**
   - Enable **sriovSupport** (single root I/O virtualization (SR-IOV))
1. EC2 to S3: **25 Gbps**
1. EC2 to EC2 (in the same or different AZs within a region):
   - **5 Gbps** of bandwidth for single-flow traffic (a flow = point-to-point network connection), or
   - **25 Gbps** of bandwidth for multi-flow traffic
1. EC2 to EC2 (Cluster Placement Group):
   - **10 Gbps** of lower-latency bandwidth for single-flow traffic, or
   - **25 Gbps** of lower-latency bandwidth for multi-flow traffic
1. Attach **2 ENIs** to maximize instance speed for an external/internal facing instance:
   - An internally-facing ENI with an MTU of **9001 bytes (cluster communication)**.
   - An externally-facing ENI with an MTU of **1500 bytes (internet communication)**.
1. You can only use high MTU values (Jumbo frames 9001 MTU) within a VPC.
1. Once the traffic leaves the VPC, use **1500 MTU**.
1. **VPN connections and traffic sent over an IGW are limited to 1500 MTU**. 
  If packets are over 1500 bytes, they are fragmented, or they are dropped if the **Don’t Fragment** flag is set in
  the IP header
1. **`tracepath`** is used to check MTU between 2 hosts; **Path MTU Discovery**; need UDP

### Placement groups
1. **Cluster placement group**: high performance, low latency, cannot span multi AZs, use first instance’s AZ.
1. **Spread placement group**: high availability, place instances across distinct underlying hardware to reduce correlated failures, max 7 running instances per AZ per group.
1. **Partition placement group**: large distributed and replicated workloads (e.g. Hadoop, Cassandra, Kafka), spread instances across logical partitions that do not share underlying hardware), max 7 partitions per AZ.

### DNS, R53
1. DNS server: **TCP/UDP/53**
1. A-record (name to ip), CNAME-record (domain name aliases), ALIAS-record (auto resolved aliases)
1. R53 automatically creates a NS (name server) record and a SOA (start of authority) record.
1. R53 assigns a unique set of **four name servers (delegation set)** to every hosted zone you create.
1. Public hosted zone
   - To create **White-Label Name Servers**, create **eight records (4 A records + 4 AAAA records)**.
1. Private hosted zone
   - Set **VPC settings to true: enableDnsHostnames, enableDnsSupport**.
   - Routing policies: Simple Routing, Failover Routing, Multivalue Answer Routing, Weighted Routing
   - To associate VPCs belonging to different accounts with a single hosted zone, 
      - Account (private hosted zone): submit a CreateVPCAssociationAuthorization request (for each VPC)
      - Account (VPC): submit an AssociateVPCWithHostedZone request.
1. To re-use nameservers across different hosted zones, create a **Reusable Delegation Set** using the **API or CLI**;
  console not supported.
1. You can't associate a reusable delegation set with a private hosted zone.
1. If you’re using custom DNS servers that are outside of your VPC and you want to use private DNS, you must use
  **custom DNS servers** on EC2 instances within your VPC.
1. R53 Resolver uses **ENIs** for inbound/outbound DNS queries between VPCs and your network.
   - On-prem to VPC -> **Inbound endpoint** (allowing R53 internal zones to be resolved)
   - VPC to on-prem -> **Outbound endpoint** (by forwarding rule)
   - The endpoints can resolve queries for VPCs in the same region **in multiple accounts using RAM**.
   - Each VPC uses the +2 resolver for the VPC, and forwards to the R53 Resolver Endpoint ENI in the “hub” VPC.
1. Create Endpoint: **2 AZ, public subnet, SG, IP address (Resolver creates a ENI for the IP in the VPC)**
1. DHCP
   - DHCP options sets: VPC-specific (VPC can have 0 or 1 DHCP options set)
   - **After you create a set of DHCP options, you can't modify them.**
   - After you associate a new set of DHCP options with a VPC, any existing instances and all new instances that you
     launch in the VPC use these options. No need to restart/relaunch the instances.
   - Options: domain-name-servers, domain-name, ntp-servers, netbios-name-servers, netbios-node-type
1. Prior to the introduction of **R53 DNS Endpoints**, the **DHCP options set** in the VPC would have to be updated with
   the **DNS forwarder** as a **custom DNS server**. This was because the forwarder had to decide if the request should
   have been sent to R53, or the on-prem DNS server.
1. Use **Squid proxy** (NAT instance/EC2) to restrict HTTP/HTTPs outbound traffic to a given set of Internet domains.

### AD 
1. **AD: port 389**
1. **Hosted AD**: replicate data from on-prem to AWS; authenticate locally without using transit bandwidth.
1. **AD Connector**: acts as a proxy to existing AD; not perform auth; may cause too much auth traffic.
1. **Simple AD**: provides IP addresses for submitting DNS queries from on-prem network to private hosted zone. 

### Resolving DNS Queries Between VPCs and Your Network
1. DNS resolution within VPC
   1. When you create a VPC, you automatically get DNS resolution within the VPC from R53 Resolver. 
   2. By default, Resolver answers DNS queries for VPC domain names such as domain names for EC2 instances or ELB load balancers. 
   3. Resolver performs recursive lookups against public name servers for all other domain names. 
   ([Source](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resolver.html))
2. On-prem to VPC DNS resolution using EC2 instance-based DNS forwarder
   1. **`Remote machine → DNS server (on-prem, example.com) → DNS Forwarder (AWS) → R53`**
3. On-prem to VPC DNS resolution using Route 53 Resolver Endpoint
   1. **`Remote machine → DNS server (on-prem) → Inbound Endpoint (VPC) → R53 Resolver (AWS)`**
4. VPC to on-prem DNS resolution using Route 53 Resolver Endpoint
   1. **`Resolver (AWS) → Outbound Endpoint (forward query with Conditional Forward rule) → DNS server (on-prem)`**
   2. A server in a VPC needs to access an on-prem client. R53 uses a Conditional Forward Rule to query the on-prem DNS server.
5. On-prem to VPC DNS resolution using Simple AD and Route 53
   1. Simple AD provides redundant and managed DNS services across AZs. 
   2. These DNS services automatically forward requests for non-authoritative domains to the VPC-provided DNS. 
      Therefore, they can be used to resolve DNS records stored in a R53 private hosted zone. 
      ([Source](https://aws.amazon.com/blogs/security/how-to-set-up-dns-resolution-between-on-premises-networks-and-aws-using-aws-directory-service-and-amazon-route-53/)) 
6. VPC to on-prem DNS resolution using Simple AD and Route 53
   ([Source](https://aws.amazon.com/blogs/security/how-to-set-up-dns-resolution-between-on-premises-networks-and-aws-using-aws-directory-service-and-amazon-route-53/))
7. On-prem to VPC DNS resolution using AWS Managed Microsoft AD
   ([Source](https://aws.amazon.com/blogs/security/how-to-set-up-dns-resolution-between-on-premises-networks-and-aws-using-aws-directory-service-and-microsoft-active-directory/))
8. VPC to on-prem DNS resolution using AWS Managed Microsoft AD
   ([Source](https://aws.amazon.com/blogs/security/how-to-set-up-dns-resolution-between-on-premises-networks-and-aws-using-aws-directory-service-and-microsoft-active-directory/))
   1. Use DHCP options set of VPC to point to Microsoft AD.
9. Centralised DNS management of hybrid cloud with R53 and Transit Gateway
   ([Source](https://aws.amazon.com/blogs/networking-and-content-delivery/centralized-dns-management-of-hybrid-cloud-with-amazon-route-53-and-aws-transit-gateway/))
10. How to add DNS filtering to your NAT instance with Squid
   ([Source](https://aws.amazon.com/blogs/security/how-to-add-dns-filtering-to-your-nat-instance-with-squid/))

### ELB
1. ALB (L7): HTTP, HTTPS; Content-based/Host-based/Path-based routing; Sticky Session, SNI, Auth
   - Listener -> Rules -> Target Groups (EC2/ECS/IP) and Health Check
   - Auto scaling does NOT apply to **IP as Target** (e.g. on-prem).
   - In order to use path-routing, need to use HTTPS (decrypt at load balancer).
1. NLB (L4): TCP, UDP, TLS; provides the ability to have static IP address; SNI
   - No SG (any traffic can reach NLB); 
   - **Use Proxy Protocol v2, source IP preserved** (only when you use EC2/ECS, not **IP as Target**)
1. CLB (L4 or 7): TCP, SSL, HTTP, HTTPS; Sticky Session
1. **How to pass the original client IP to the backend for non web application?** 
   - Only NLBs supports source IP preserving for **Non-HTTP** applications on EC2 instances.
   - ALBs/CLBs (connect to instances with private Load Balancer IP) do not support source IP preserving.
   - For HTTP services on your instances, you can get the client IP with **X-Forwarded-For header**. 
   - For non-HTTP (e.g. SMTP) services, enable **Proxy Protocol** and implement Proxy-protocol in your backend service.
1. SSL negotiation: change the **security policy** on the ELB to disable vulnerable protocols and ciphers.
1. SSL Offload: SSL cert needs only live on the Load Balancer, and it performs the encryption and decryption operations.
  This will free up resources on the smaller instances.
1. Client->CLB->Backend (TCP/TCP): put LB in passthrough mode; not require SSL cert on LB.
1. CLB cannot validate a client side certificate, so it must be passed through as standard TCP on **port 443** to let
  the EC2 instance handle the validation.
1. Health check is per Target Group.
1. CloudWatch metrics:
   - ActiveFlowCount, NewFlowCount, ProcessedBytes
   - TCP_Client_Reset_Count, TCP_ELB_ResetCount, TCP_TargetResetCount
   - HealthyHostCount, UnhealthyHostCount

### CloudHSM 
1. When you create a CloudHSM cluster with more than one HSM, you automatically get load balancing.
1. CloudHSM instances **SSL/TLS offload** with ELB: **NLB, listener 443, target 443**.

### CloudFront, WAF
1. You can use Lambda@Edge functionality of CloudFront to inject the security headers: e.g. X-XSS-Protection
1. CloudFront: Global Edge Location -> Region Edge Cache Location -> Origin server
1. **For PCI DSS: configure the CloudFront Cache Behavior to redirect HTTP requests to HTTPS and to forward request to the origin via the Amazon private network. (Cannot rely on Match Viewer).**
1. **WAF** can be attached to your **CloudFront, ALB, API Gateway** to dynamically detect and prevent attacks.
   - CloudFront + WAF
     - It's ineffective if origin servers can be attacked directly, bypassing CloudFront.
     - Solution: Configure CloudFront to use a custom header and configure an AWS WAF rule on the origins ALB to accept only traffic that contains that header.
     - Because CloudFront distributions are not associated with SG, nor are fixed IPs available.

### WorkSpaces
1. **443/TCP (client/reg/auth), 4172/TCP/UDP (streaming), 80/TCP/UDP (init conn), 53 /UDP (DNS)**
1. If your WorkSpaces users are unable to authenticate, check if **port 389** is open to your AD server.
1. **minimum 1200 MTU**.
1. Workspaces uses AD; options: Microsoft AD, AD connector, Simple AD; requires **2 subnets in Multi-AZs**.
1. Configure a VPC with 1 public subnet (NAT Gateway) and 2 private subnets (WorkSpaces).
1. Each WorkSpace has two ENIs
   1. a network interface for management and streaming (eth0) and
   2. a primary network interface (eth1).
   3. The primary network interface has an IP address provided by your VPC, from the same subnets used by the directory. This ensures that traffic from your WorkSpace can easily reach the directory.
1. **VPC SG** enables to limit access to resources in the network or the Internet from the WorkSpaces.
1. **IP Access Control Group** allows trusted IP addresses to access the WorkSpaces (as a virtual firewall).

### AppStream 2.0
1. AppStream: **443 (TCP)** and **1400-1499 (TCP)**
1. AppStream 2.0 requires Internet connectivity to authenticate users and deliver the web assets. 
1. How do I use my DX, AWS VPN, or other VPN tunnel to stream my applications?
   1. Configure a VPC with private subnets and a NAT Gateway (recommended)
   2. Create a **VPC interface endpoint** for AppStream in the **same VPC** as your DX, VPN, or VPN tunnel.
   3. The interface endpoint maintains the streaming traffic within your VPC. You must allow:
        1. **`.amazonappstream.com` (Session Gateway)** on the network from which users initiate access to the
           streaming instances.
        2. **`appstream2.<region>.aws.amazon.com`** to enable user authentication.
   4. **SG** associated with the interface endpoint must allow inbound access to **443 (TCP)** and **1400-1499 (TCP)**
       from the IP range from which your users connect.
   5. **NACL** must allow outbound from ephemeral ports **1024-65535 (TCP)** to the IP range of users.
1. 3 options for Internet access:
   1. Private subnets + NAT gateway + IGW
   2. Public subnet + IGW (enable Default Internet Access)
   3. Default VPC with public subnet and SG (enable Default Internet Access)
1. When **Default Internet Access** is enabled, max **100 fleet instances** is supported. 
  If your deployment must support more than 100 concurrent users, use the **NAT gateway** configuration instead.
