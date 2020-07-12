# VPC Flow Log examples
```
<version> <account-id> <interface-id> <srcaddr> <dstaddr> <srcport> <dstport> <protocol> <packets> <bytes> <start> <end> <action> <log-status>
```

### Case 1
Ping from your home computer (203.0.113.12) to your instance (172.31.16.139).  ([Source-1](
https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-records-examples.html))
- SG inbound rules allow ICMP traffic but the outbound rules do not allow ICMP traffic. Because SGs are stateful, the
  response ping from your instance is allowed. 
- NACL permits inbound ICMP traffic but does not permit outbound ICMP traffic. Because NACLs are stateless, the
  response ping is dropped and does not reach your home computer. 
- In a default flow log, this is displayed as two flow log records:
   - An ACCEPT record for the originating ping that was allowed by both the NACL and SG, and therefore was allowed to
     reach your instance.
   - A REJECT record for the response ping that the NACL denied.
```
2 123456789010 eni-1235b8ca123456789 203.0.113.12 172.31.16.139 0 0 1 4 336 1432917027 1432917142 ACCEPT OK
2 123456789010 eni-1235b8ca123456789 172.31.16.139 203.0.113.12 0 0 1 4 336 1432917094 1432917142 REJECT OK
```

### Case 2 
If your network ACL permits outbound ICMP traffic, the flow log displays two ACCEPT records (one for the originating
ping and one for the response ping).  ([Source-1](
https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-records-examples.html))

```
2 123456789010 eni-1235b8ca123456789 203.0.113.12 172.31.16.139 0 0 1 4 336 1432917027 1432917142 ACCEPT OK
2 123456789010 eni-1235b8ca123456789 172.31.16.139 203.0.113.12 0 0 1 4 336 1432917094 1432917142 ACCEPT OK
```

### Case 3
If your security group denies inbound ICMP traffic, the flow log displays a single REJECT record, because the traffic
was not permitted to reach your instance. ([Source-1](
https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs-records-examples.html))

```
2 123456789010 eni-1235b8ca123456789 203.0.113.12 172.31.16.139 0 0 1 4 336 1432917027 1432917142 REJECT OK
```

### Case 4
If you added an NACL outbound rule to deny ICMP echo response from the EC2 to my external machine, there were 3 records
as shown below. The display time on CloudWatch could be the same for all 3 of them. ([Source-2](
https://acloud.guru/forums/aws-certified-advanced-networking-specialty/discussion/-LbS0vrAliwhoQa2wp23/Is%20the%20flowlog%20entries%20explanation%20at%203:10%20correct%3F
))

```
2 9xxx eni-xxx 123.45.67.89 172.31.44.17 0 0 1 2 120 1554186843 1554186883 ACCEPT OK
2 9xxx eni-xxx 172.31.44.17 123.45.67.89 0 0 1 1 60 1554186843 1554186883 ACCEPT OK
2 9xxx eni-xxx 172.31.44.17 123.45.67.89 0 0 1 1 60 1554186843 1554186883 REJECT OK
```
The first two pings were bundled into one record (120 bytes). That is because of the Flow Log capture window. The
responses have the same display time on CloudWatch, but the ACCEPT corresponds to my first Ping, and the REJECT
corresponds to my second ping after I modified the NACL.
