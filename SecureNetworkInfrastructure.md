# Secure Network Infrastructure

Table of Contents

- [Security Groups](#security-groups)
- [NACL (Network Access Control List)](#nacl-network-access-control-list)
- [VPC Endpoints (Gateway Endpoints and Interface Endpoints)](#vpc-endpoints-gateway-endpoints-and-interface-endpoints)
- [NAT Gateways vs. NAT Instances](#nat-gateways-vs-nat-instances)
- [Egress-Only: NAT Instance/Gateway for IPv4 vs. Engress-Only Internet Gateway for IPv6](#egress-only-nat-instancegateway-for-ipv4-vs-engress-only-internet-gateway-for-ipv6)


## Security Groups

1. Specify the accountID/sgID to reference a remote Security Group.


## NACL (Network Access Control List)


## VPC Endpoints (Gateway Endpoints and Interface Endpoints)

VPC Endpoints allow access to public AWS services, or service provided by 3rd parties, without requiring an Internet
Gateway to be attached to the VPC, or any NAT instance/gateway.
It means your network communications no longer have to flow over the public internet to reach the public interfaces of
AWS services such as S3, API Gateways, etc.

### Gateway Endpoints

- Attach Endpoint policy; not SG
- Region specific; need to be in the same region.
- Support AWS Public Services
- The following services are supported:
  - S3
  - DynamoDB
- A **Gateway Endpoint** requires a route in the route tables for any subnets where the gateway will be used, for
  traffic destined to a supported AWS services.
- The destination is a "prefix list" (always prefixed with "pl-"), which is virtual entry which represents a service
  in a region.
- E.g. Route: `Destination=pl-1a2b3c4d, Target=vpce-11bb22cc (VPCE-ID)` 
- **Endpoint policies** will be applied to the endpoint to limit the functionality of the endpoint.
   ```
   "Resource": ["arn:aws:s3:::sharedpics", "arn:aws:s3:::sharedpics/*"]
   ```
- **Resource policy** can be utilised to limit the resource to one or more VPC endpoints.  
- You cannot use `aws:SourceIp` condition, use `aws:sourceVpce` instead.
   ```
   "Principal": "*",
   "Action": "s3:*",
   "Effect": "Deny",
   "Resource": ["arn:aws:s3:::sharedpics", "arn:aws:s3:::sharedpics/*"],
   "Condition": {
     "StringNotEquals": { "aws:sourceVpce": "vpce-1a2b3c4d" }
   }
   ```

### Interface Endpoints (powered by AWS PrivateLink)

- An **Interface Endpoint** is an ENI (Elastic Network Interface) with a private IP address that serves as an entry
  point for traffic destined to a supported service.
- Attach SG and allow referencing other SGs.
- AZ specific; can create interface for each AZ.
- Support TCP traffic only
- Support IPv4 only
- Can be accessed through Direct Connect connection.
- Can be accessed through same-region VPC peering connection from C5, i3.metal, R5, R5D, Z1D instance types only.
- Cannot be accessed through an inter-region (different regions) VPC peering connection, or an AWS VPN connection.
- Inject interface into your VPC.
- Do not support endpoint policies like that of Gateway Endpoints.
- Support AWS Public Services and Third Party Services.
- The following services are supported:
  - CloudWatch Logs
  - EC2 API
  - Kinesis Data Streams
  - SNS
  - KMS
  - Service Catalog
  - Systems Manager
  - ELB API
  - Endpoint services hosted by other AWS accounts 
- Case 0:   Not using Interface Endpoint
  - Instance R (instance on the right) accesses the SNS service through Internet Gateway.
  - DNS: sns.us-east-1.amazonaws.com
  - IP:  52.*.*.* (public IP)
- Case 1:   Using Interface Endpoint
  - Instance L (instance on the left) accesses SNS service using endpoint specific name through the Interface Endpoint.
  - DNS: vpce-<id>.sns.region.amazonaws.com
  - IP:  10.*.*.* (private IP)
- Case 2:  Using Interface Endpoint with Private DNA Name
  - If we want instance R to use Interface Endpoint but we cannot change the default DNS it is referenced.
  - **Enable Private DNA Name (when creating new Interface Endpoint)****
    - To use private DNS names, ensure that the attributes Enable DNS hostnames and Enable DNS Support are set to true
      for your VPC.
  - DNS: sns.us-east-1.amazonaws.com (still)
  - IP: 10.*.*.* (private IP).
- Limitations
  - May not be available in all AZs.
  - Support TCP traffic only. 
  - Support IPv4 traffic only.
  - Interface endpoints can be accessed through Direct Connect connection.
  - Interface endpoints can be accessed through intra-region (same region) VPC peering connection from C5, i3.metal, R5,
    R5D, Z1D instance types only.
  - Interface endpoints cannot be accessed through an inter-region (different regions) VPC peering connection, or an
    AWS VPN connection.


## NAT Gateways vs. NAT Instances

### NAT Gateways (Network Address Translation Gateways)
- NAT Gateways are a new "managed" service to complement NAT Instances (older).
- NAT Gateways provide internet access for AWS services (e.g. EC2, Lambda) within private subnets.
- NAT Gateways are located in a single subnet and utilize Elastic IPs.
- NAT Gateways cannot have security groups associated with them.
- NAT Gateways operate in subnets which can have NACLs, and the source (e.g. EC2) subnets can have NACLs.
- E.g. Route: `Destination=0.0.0.0/0, Target=nat-1111222233334444 (NAT-ID)` 
- No need to to concern ingress traffic, just manage outgoing traffic.
- Cannot ssh to a NAT Gateway.
- Cannot do port forward
- Create a NAT Gateway for more than one AZ.

### NAT Instances (Network Address Translation Instances)
- NAT Instance is a pre-configured EC2 instance performing the same function as NAT Gateway.
- NAT Instances could be secured with NACL (on the private or NAT subnet) and security groups on the NAT instance and
  source service (e.g. EC2).


## Egress-Only: NAT Instance/Gateway for IPv4 vs. Engress-Only Internet Gateway for IPv6

1. With **IPv4**, all AWS resources have a private IP. Some can be provided with a public IP and connectivity, using an
   **Internet Gateway (IGW)**.  With **IPv4** a **NAT Instance/Gateway** can be utilized to provide **egress-only
   access**.
   
2. **IPv6** addressing is globally unique and publicly routable.  Supported resources in AWS are all publicly
  addressable, so a **NAT Gateway** isn’t an option.

3. An **Egress-Only Internet Gateway (EIGW)** provides a feature-limited Internet Gateway, specifically for **IPv6**,
   and only allowing outbound connections and return traffic (**stateful**).
   No incoming IPv6 connections can be initiated to VPC resources using Egress-Only Gateway.
   
   - The VPC router via a route table needs to have a IPv6 default route (or a specific one) added.
   - E.g. Route: `Destination= ::/0, Target= eigw-1111222233334444 (EIGW-ID)`
   - Note: IPv6 format: (`::/0`), IPv4 format: (`0.0.0.0/0`)
   - Optionally you can have a route for IPv4 traffic via a NAT and IGW combination.
   - You cannot restrict connections based on DNS, IP or authentication on the Gateway itself.
