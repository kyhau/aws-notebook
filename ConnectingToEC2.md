# Connecting to EC2

## EC2 Instance Connect vs. Session Manager
See also [Different between EC2 Instance Connect and Session Manager](
https://medium.com/@ystataws/different-between-ec2-instance-connect-and-session-manager-c1a0b110b474).

1. **EC2 Instance Connect**
    1. Role: No need
    2. S.G.: Need
    3. IPv4: Yes
    4. Browser: Yes, Safari not working (as of 2019-08-15)
    5. CloudTrail: Yes
2. **Session Manager**
    1. Role: Need
    2. S.G.: No need
    3. IPv4: *Depends***
    4. Browser: Yes
    5. CloudTrail: Yes
    6. Beware of active session.  A session remains active even you revoke an IAM role.
    7. Need to tighten the CloudWatch Log of session manager - a user can `cat` a sensitive file and all
       content goes to the session log.
    8. Make sure correct permission in place to control who can see the log, no one can change the logs,
       and who can change the log group and roles.
   
**Notes**
- IPv4
    - If your EC2 is in public subnet with IPv4, then it can talk to SSM through internet. 
    - If your EC2 is in private subnet, then you either need to have NAT Gate/Instance or VPC endpoints setup so EC2
      can talk to SSM through either of them.
- Security Group:
    - Session Manager does not require any inbound Security Group configuration for the EC2. 
      Cause it is the agent installed on EC2 communicating to SSM which is outbound flow and it is open to all by
      default.
