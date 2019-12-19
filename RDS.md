# RDS

- RDS now allows you to easily stop and start database instances. (See [Source](
https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_StopInstance.html))
    - While a database instance is stopped, RDS does not delete any of your automatic backups or transaction logs.
      This means you can do a point-in-time restore to any point within your specified automated backup retention
      window, even after an instance is started. 
    - Starting an instance restores it to the same configuration as it had when stopped, including its endpoint, DB
      parameter group, security group, and option group membership.
    - You can stop an instance for up to **7 days** at a time. After 7 days, it will be automatically started.
    - The stop/start feature is available for database instances running in a **Single-AZ deployment** which are not
      part of a Read Replica (both source and replica) configuration.
