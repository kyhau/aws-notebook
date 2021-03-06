# Logging and Monitoring

Topics

- [AWS Config](#aws-config)
- [Trusted Advisor](#trusted-advisor)
- [CloudTrail](#cloudtrail)
- [CloudWatch](#cloudwatch)
- [CloudWatch Agent](#cloudwatch-agent)
- [CloudWatch Buses](#cloudwatch-buses)
- [CloudWatch Insights](#cloudwatch-insights)
- [SSM (Systems Manager)](#ssm-systems-manager)
- [Inspector](#inspector-inspector-agent)
- [Packet Capture Agent](#packet-capture-agent)
- [VPC Flow Logs](#vpc-flow-logs)
- [VPC Traffic Mirroring](#vpc-traffic-mirroring)
- [DNS Logs](#dns-logs)
- [S3 server access logging for bucket](#s3-server-access-logging-for-bucket)
- [Macie](#macie)
- [GuardDuty](#guardduty)
- [RDS Performance Insights](#rds-performance-insights)

---
## AWS Config
- Record and monitor the configurations of AWS resources.
  - Retrieve current configurations of resources in your account.
  - Retrieve historical configurations.
  - View relationships between resources (e.g. members of a security group).
- Evaluate resource configurations for desired settings.
  - Compare the user's IAM permissions before and after the incident.
  - Check whether logging is enabled for buckets.
- Get notification for creation, deletions, and modifications.
- Get notification when a resource violates configuration rules.
- Security analysis e.g. auditing historical records of IAM policies, security group configurations.

## Trusted Advisor
- Trusted Advisor provides account level recommendations on improvements under Cost Optimization, Performance,
  Security and Fault Tolerance areas.
- Trusted Advisor warns us of security groups with a source of 0.0.0.0/0. 
- Trusted Advisor warns us if an RDS security group is overly permissive.

## CloudTrail
- CloudTrail logs all API activities (including Console, CLI, API/SDK calls).
- CloudTrail is enabled when your account is created.
- Logs are encrypted in **SSE-S3 by default**, can be changed to SSE-KMS.
- Entries can be viewed using the **Event History** in **past 90 days**.
- Trail logs can be sent to S3 bucket of choice and even prefixed (folders).
- Single-region or multi-region trails can be configured.
- Trails can make multi-account logging possible.
- Management events: user login events, configuring security, setting up logging.
  - *CloudTrail does not log configurations or application logs.*
  - *CloudTrail does not utilize log groups; CloudWatch does.*
  - *Use CloudTrail to record KMS API calls. KMS is not one of the services that can send logs to CloudWatch Logs.*

## CloudWatch
- CloudWatch monitors web application logs to for malicious activity.
- Retention settings: 1 day to never expire
- Sources: CloudTrail, VPC Flow Logs, CloudWatch Agent, DNS Logs (from Route53)
- [Cross-Account Cross-Region CloudWatch Console](
  https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Cross-Account-Cross-Region.html)

## CloudWatch Agent
- CloudWatch/any AWS service cannot monitor your EC2 filesystems without an agent installed.
- CloudWatch Agent collects additional metrics and logs from EC2 (and on-premise). 
- E.g. memory, disk-use percentages, and swap file usage.
- It can also collect logs from the application.

## CloudWatch Buses
- CloudWatch Bus allows different AWS accounts to share CloudWatch Events.
- CloudWatch Bus can collect events from all your accounts together in one account.

## CloudWatch Insights
- [Anomaly Detection](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Anomaly_Detection.html)
- [Application Insights for .NET and SQL Server](
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/appinsights-what-is.html)
- [Container Insights for ECS and EKS](
  https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/deploy-container-insights.html)

## SSM (Systems Manager)
- SSM periodically scans EC2 instances, or on-premise servers/VMs, retrieving details of installed applications, AWS
  components, network configs, windows updates, detailed information on an instance/VM, details on running services,
  windows roles, and optional custom data SSM can collect on your behalf.
- Use SSM Run Command to get information from the OS level of an instance.
- Use SSM Patch Manager to generate the report of out of compliance instances/servers.
- Use SSM Patch Manager to install the missing patches.
- Security groups do NOT govern the communication between SSM and the instances.
- If the SSM Run Command is not executing on some of the instances:
  - Ensure the SSM agent is running on the target machine.
  - Check the `/var/log/amazon/ssm/errors.log` file.

## Inspector (Inspector Agent)
- Inspector identifies potential security issues and analyzes behaviour of your AWS resources.
- Use Inspector to obtain the list of vulnerabilities.
- Inspector provides detailed recommendations for resolving issues.
- Use SSM Run Command or manually install Inspector Agent.

## Packet Capture Agent
- Use SSM Run command feature to install a packet capture agent on all EC2 instances, configure the software to store
  the capture logs in a central location.
- **Packet sniffing vs. VPC Flow Logs**
   - Sniffed packets are captured in their entirety and (unlike VPC Flow Logs) can be inspected at a data level - 
     providing they are not encrypted.
   - VPC Flow Logs does not allow traffic capture, only metadata.

## VPC Flow Logs
- VPC Flow Logs can help you see if there are rejects or responses to your network traffic. It will help to determine
  where the traffic is failing.
- Can be assigned to a VPC, a subnet or an ENI.
- *VPC Flow Logs contain traffic metadata only - it won't show content.*

## VPC Traffic Mirroring
- See [New – VPC Traffic Mirroring – Capture & Inspect Network Traffic](
https://aws.amazon.com/blogs/aws/new-vpc-traffic-mirroring/)

## DNS Logs
- DNS Query Logs can be enabled on Route53 hosted zones and sent to CloudWatch.
- These logs can be used to determine when there is a DNS problem in an application.
- These logs are only available for hosted zones where Route53 is the endpoint (no outside hosting).
- _The logs are not available for private hosted zones._

## S3 Server access logging for bucket
- You want to track requests for access to a particular S3 bucket.
- The Log Delivery group must be granted write permission on the target bucket.
- *Logs are delivered on a "best effort" basis (NOT near-real-time logging).*

## Macie
- Macie helps you protect your data in S3 by helping you classify what data you have, the value that data has to the
  business, and the behavior associated with access to that data. 
- Both GuardDuty and Macie services incorporate user behavior analysis, machine learning, and anomaly detection to
  detect threats in their respective categories.

## GuardDuty
- GuardDuty provides broad protection of your AWS accounts, workloads, and data by helping to identify threats such
  as attacker reconnaissance, instance compromise, and account compromise. 
- GuardDuty analyzes continuous streams of metadata generated from your account and network activity found in
  CloudTrail Events, VPC Flow Logs, and DNS Logs.
- GuardDuty is a threat detection service and will not report on the configuration of security groups.

## RDS Performance Insights
- [RDS Performance Insights](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PerfInsights.html)