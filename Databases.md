# Focus

- [RDS](#rds)
- [Parameter Groups, DB Cluster Parameter Groups, Option Groups](#parameter-groups-db-cluster-parameter-groups-option-groups)
- [Monitoring, Logging and Alerting](#monitoring-logging-and-alerting)
- [Storage volume of managed services: Aurora, DocumentDB, Neptune](#storage-volume-of-managed-services-aurora-documentdb-neptune)
- [Aurora (Single-Master Clusters)](#aurora-single-master-clusters)
- [Aurora (Multi-Master Clusters)](#aurora-multi-master-clusters)
- [Aurora (Serverless Clusters V1)](#aurora-serverless-clusters-v1)
- [DynamoDB](#dynamodb)
- [EC2 Instance Databases](#ec2-instance-databases)
- [Migrating Databases](#migrating-databases)
- [ElastiCache for Memcached vs. Redis](#elasticache-for-memcached-vs.-redis)
- [ElastiCache for Memcached](#elasticache-for-memcached)
- [ElastiCache for Redis](#elasticache-for-redis)
- [Redshift](#redshift)
- [DocumentDB](#documentdb)
- [Neptune](#neptune)
- [CloudFormation, Secret Manager and SSM parameter store](#cloudformation-secret-manager-and-ssm-parameter-store)
- [IAM Database Authentication, Encryption](#iam-database-authentication-encryption)
- [Network Troubleshooting](#network-troubleshooting)
- [Choosing the right database technology](#choosing-the-right-database-technology)
- [Keyspaces](#keyspaces)
- [QLDB](#qldb)

---
## RDS

1.  RDS: **20 GiB - 64 TiB** of storage (except **max 16 TiB** for SQL Server), Interface: **SQL**

2.  Availability, reliability, fault tolerance, upgrade

    1. Multi-AZ: Automatic failover to **standby** (routing DNS entries), complete within **1 - 2 minutes**.
        ([Ref](https://aws.amazon.com/rds/features/multi-az/))

    2. Single-AZ: Need manual point-in-time-restore; can take **several hours** to complete.
        ([Ref](https://aws.amazon.com/rds/features/multi-az/))

    3. Hardware maintenance
        ([Ref](https://aws.amazon.com/premiumsupport/knowledge-center/rds-required-maintenance/))

        1. Multi-AZ deployments are unavailable for the time it takes
            the instance to failover (**1 - 2 min**) if the AZ is
            affected by the maintenance. If only the secondary AZ is
            affected, then there is no failover or downtime.

        2. Single-AZ deployments are unavailable for **a few minutes**.

    4. Auto OS/system maintenance:

        1. For Multi-AZ deployments, OS maintenance is applied to the
            **standby** first, then the instance fails over, and then
            the **primary instance** is updated. The downtime is
            during failover.

    5. DB engine maintenance (optional auto minor version upgrade,
        manual major engine version upgrade):

        1. Even if your RDS DB instance uses a Multi-AZ deployment,
            both the **primary** and **standby** DB instances are
            upgraded at the same time. This causes downtime until the
            upgrade is complete, and the duration of the downtime
            varies based on the size of your DB instance.

    6. A **Read replica** can be manually promoted to a standalone
        database instance.

    7. RDS allows a **standby** instance in another Region. Combining
        this feature with **Multi-AZ read replicas** further improves
        availability, latency, and disaster recovery. Choose an
        instance class optimized for provisioned IOPS for consistent
        performance in the event of a failover to standby.

    8. You can also promote a **cross-region read replica** to be the
        primary instance.

    9. Put an SQS queue in front of the database. If the database is
        unreachable, messages are stored in a queue. When the database
        becomes available, messages in the queue are resubmitted.

3.  Multi-AZ and automatic failover (optional)
    ([multi-az](https://aws.amazon.com/rds/details/multi-az/))
    ([RDS FAQs](https://aws.amazon.com/rds/faqs/))

    1. **Synchronous** physical replication to keep data on the
        **standby** in different AZ up-to-date with the **primary**.

    2. Only the **primary** instance is active.

    3. Use Provisioned IOPS with Multi-AZ instances for fast,
        predictable, and consistent throughput performance.

    4. Automatic failover to **standby** (routing DNS entries),
        complete within **1 - 2 minutes**.

        1. System upgrades and DB Instance scaling are applied first on
            the **standby**, prior to automatic failover.

        2. Failover time can be affected by whether large uncommitted
            transactions must be recovered; use smaller transactions,
            and adequately large instance types for best results.

        3. After the failover, RDS automatically rebuilds a
             **standby** server in another AZ.

    5. Single-AZ: Need manual point-in-time-restore; can take **several
        hours** to complete, and any data updates that occurred after
        the latest restorable time (last **5 mins**) will not be
        available.
        ([Ref](https://aws.amazon.com/rds/features/multi-az/))

    6. Use **RDS Event Notification** to monitor failovers.
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_Events.html)).

    7. If your application caches DNS values, setting **TTL** (time to
        live) to less than **30 secs** is a good practice in case
        there is a failover, where the IP address might change and the
        cached value might no longer be in service.

    8. You might experience connection issues after a failover if the
        subnet of the primary instance has different traffic-routing
        rules than the subnet of the standby instance. Be sure the
        subnets in your **database subnet group** have consistent
        routing rules.

    1. For **SQL Server**, do not enable the following modes as they
        turn off transaction logging, which is required for Multi-AZ
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html#CHAP_BestPractices.SQLServer))

        1. Simple recover mode
        2. Offline mode
        3. Read-only mode

4.  Multi-Region = cross-region read replicas

5.  Scaling - Read
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html))
    ([read-replicas](https://aws.amazon.com/rds/details/read-replicas/))

    1. RDS creates a second DB instance using a **snapshot** of the
        source DB instance. It then uses the engines' native
        asynchronous replication to update the read replica whenever
        there is a change to the source DB instance.

    2. Max **5** read replicas. All read replicas are accessible and
        can be used for **read** scaling.

    3. No backups configured by default.

    4. Database engine version upgrade is independent from the **source
        instance**.

    5. Read replicas **need not** use the same type of **storage** as
        their master DB Instances.

    6. You can set up a read replica with its own **standby** instance
        in different AZ. This functionality complements the
        synchronous replication, automatic failure detection, and
        failover provided with Multi-AZ deployments.

    7. A Read replica can be manually promoted to a standalone database
        instance.

    8. Adding **read replicas** to the database will improve
        performance and horizontally scale the reads, but it will
        **NOT** improve the reliability and fault tolerance of the
        architecture.

    9. CloudWatch ReplicaLag metric: 0 = replica has caught up to the
        source DB instance; -1 = replication not active

    10. To distribute the read traffic to the RDS read replicas, use R53
        weighted record sets (**Weighted routing policy)**.

        1. Within a R53 hosted zone, create individual record sets for
            each **Read Replica Endpoint** (or **Reader Endpoint**)
            associated with the read replicas and give them the same
            weight. Then, direct requests to the endpoint of the
            record set.

        2. RDS does not support **reader endpoint** with automatic load
            balancing of read traffic. This feature is only available
            for Aurora.

        3. NLB supports only IP addresses and instance IDs as target
             types (not RDS endpoints).

        4. ALB supports only IP addresses, instance IDs and Lambda
            functions as target types (not RDS endpoints).

    11. **Second-tier read replica**
        ([Ref](https://aws.amazon.com/rds/faqs/))

        1. Supported (Aurora, RDS MySQL/MariaDB); not supported (RDS
            PostgreSQL/Oracle/SQL Server).

        2. You can create a second-tier Read Replica from an existing
            first-tier Read Replica. By creating a second-tier Read
            Replica, you may be able to move some of the replication
            load from the master database instance to a first-tier
            Read Replica.
            ([Ref](http://aws.amazon.com/rds/faqs/#mysql))

        3. A second-tier Read Replica may lag further behind the
             master because of additional replication latency.

    12. Scalability, can be within an AZ, Cross-AZ, or Cross-Region.

    13. **Cross-region read replicas**
        ([RDS-FAQs](https://aws.amazon.com/rds/faqs/),
        [PostgreSQL](https://aws.amazon.com/blogs/database/best-practices-for-amazon-rds-for-postgresql-cross-region-read-replicas/),
        [MySQL](https://aws.amazon.com/blogs/aws/cross-region-read-replicas-for-amazon-rds-for-mysql/),
        [SQL
        Server](https://aws.amazon.com/blogs/database/cross-region-disaster-recovery-of-amazon-rds-for-sql-server/),
        [Oracle](https://aws.amazon.com/blogs/database/cross-region-automatic-disaster-recovery-on-amazon-rds-for-oracle-database-using-db-snapshots-and-aws-lambda/))

        1. RDS (except **RDS SQL Server**) supports **cross-region read
            replicas** (with network latency).

        2. For optimal performance, your replica instance must be the
            same class (or above) and same storage type as the source
            instance. Replica instances not only replay similar write
            activity as the master, but also serve additional read
            workloads.

        3. Automated backups can be taken in **each region**.

        4. Each region can have a Multi-AZ deployment.

        5.  Database engine version upgrade is independent in **each
            region**.

    14. If you have performance issue caused by increased read activity
        on your RDS MySQL deployed in Multi-AZ:

        1. Read Replicas may not be the most performant solution. Read
            Replica performance is dependent on the instance size.

        2. Deploy a **ElastiCache cluster** in front of the RDS DB
            instance. ElastiCache offers a better read performance
            solution (provides sub-millisecond response for read
            queries).

    15. RDS MySQL allows you to add table indexes directly to Read
        replicas, without those indexes being present on the master.

6.  Scaling - Storage
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIOPS.StorageTypes.html#USER_PIOPS.Autoscaling))

    1. With storage autoscaling enabled, RDS starts a storage
        modification when:

        1. Free available space < 10% of the allocated storage.
        2. The low-storage condition lasts at least **5 minutes**.
        3. At least **6 hours** since the last storage modification.

    2. The additional storage is in increments of whichever of the
        following is greater:

        1. 5 GiB
        2. 10% of currently allocated storage
        3. Storage growth prediction for **7 hours** based on the
             **FreeStorageSpace** metrics change in the past hour.

    3. Enable **storage auto scaling** with **ModifyDBInstance**
        (modify-db-instance) action.

        1. If **--max-allocated-storage** (GB) is greater than
            **--allocated-storage**, storage auto scaling is turned
            on.

        2. If **--max-allocated-storage ==** **--allocated-storage**,
            storage auto scaling is turned off.

    4. If you start a storage scaling operation at the same time that
        RDS starts an autoscaling operation, your storage modification
        takes precedence. The autoscaling operation is canceled.

    5. Autoscaling cannot be used with **magnetic** storage.

    6. Autoscaling does not occur if the maximum storage threshold
        would be exceeded by the storage increment.

    7. Autoscaling cannot completely prevent storage-full situations
        for large data loads, because further storage modifications
        cannot be made until **6 hours** after storage optimization
        has completed on the instance. If you perform a large data
        load, and autoscaling does not provide enough space, the
        database might remain in the storage-full state for several
        hours. This can harm the database.

    8. When you **clone** an RDS DB instance, the **storage
        autoscaling** setting is not automatically inherited by the
        cloned instance. The new DB instance has the same amount of
        allocated storage as the original instance.

       1. The **database cloning** feature is only available in Aurora, not RDS.

    9. Autoscaling cannot be used with the following
        previous-generation instance classes that have less than **6
        TiB** of orderable storage: db.m3.large, db.m3.xlarge, and
        db.m3.2xlarge.

7.  When performing an upgrade, an `InsufficientDBInstanceCapacity` error is
    returned, and you are unable to modify the RDS instance:

    1. Retry the request with a different database instance class.

    2. Retry the request without specifying an explicit AZ.

8.  To prevent accidental deletion of RDS databases, either

    1. Set the **deletion policy** of the database resource to
        **retain**.

    2. Enable **termination protection** on the CloudFormation stack
        (disabled by default) with any status except
        **DELETE_IN_PROGRESS** or **DELETE_COMPLETE**.

9.  Deleting a RDS instance

    1. A **cluster-level snapshot** should be in place before deleting
        an RDS instance.

        1. When deleting an RDS instance using CLI, the following error
            is encountered:

                An error occurred (InvalidParameterCombination) when calling the
                DeleteDBInstance operation: FinalDBSnapshotIdentifier cannot
                be specified when deleting a cluster instance.

        2. Use the **--skip-final-snapshot** flag in the CLI **delete**
            command to skip the final snapshot.

    2. When the source database is **deleted**, what happens to the read
        replicas?
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html))

        1. All read replicas are promoted.

        2. For **MariaDB**, **MySQL**, and **Oracle** RDS instances, when
            the source database is deleted, read replicas in the same
            region and cross-region read replicas are **promoted**.

        3. For **RDS PostgreSQL instance**s, when the source database is
                deleted, read replicas in the same region are **promoted**,
                and cross-region read replicas are set to replication status
                "**terminated**".

10. Stopping a RDS database

    1. You can stop a DB instance for up to **7 days**. After 7 days,
        the DB instance is automatically started. This is required to
        perform the maintenance updates.

    2. You can stop and start a DB instance whether it is configured
        for a single-AZ or for Multi-AZ, for database engines that
        support Multi-AZ deployments.

        1. You cannot stop an RDS for **SQL Server** DB instance in a
            Multi-AZ configuration.

    3. You cannot stop a DB instance that has a read replica, or that
        is a read replica.

    4. You cannot modify a stopped DB instance.

    5. You cannot delete a **DB parameter group** or an **option
        group** that is associated with a stopped DB instance.

11. Encryption: If enabled, applies to underlying storage for DB
    clusters, automated backups, read replicas, and snapshots.

    1. Enabling encryption for existing RDS and Aurora with minimal
        downtime
        ([rds-encryption](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html))
        ([rds-encryption-kms](https://aws.amazon.com/blogs/database/securing-data-in-amazon-rds-using-aws-kms-encryption/))

    2. Encrypting an existing DB Instance is not supported. Two options to
        encrypt an existing DB instance:

        1. You can create a snapshot of your DB instance, and then create an
            encrypted copy of that snapshot. You can then restore a DB
            instance from the encrypted snapshots.
            ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.Encryption.html#Overview.Encryption.Limitations))

        2. You can create a new DB Instance with encryption enabled and migrate
            your data into it.

    2. An AWS account has a different **default CMK for each Region**.

    3. Once an encrypted DB instance is created, you cannot change the type
        of CMK used by that DB instance.

    4. The primary DB instance and its read replicas in the same Region
        must be encrypted with the same CMK.

    5. If the primary DB instance and read replica are in different
        Regions, you encrypt using the CMK for that Region.

    6. You cannot have an encrypted read replica of an unencrypted DB
        instance or an unencrypted read replica of an encrypted DB
        instance.

    7. You cannot restore an unencrypted backup or snapshot to an encrypted
        DB instance.

    8. RDS has support for **Transparent Data Encryption (TDE)** for **SQL
        Server** and **Oracle** (enabled with **option group**). This
        encryption mode allows the server to automatically encrypt data
        **before** it is written to storage.

        1. You cannot share Oracle or Microsoft SQL Server snapshots that
            are encrypted using TDE.

12. Automatic Backups

    1. Backup retention period: **0 - 35 days**. 0 = disable automated
        backup.

    2. Disabling automatic backups for a DB instance deletes all
        existing automated backups for the instance.

    3. Default backup retention period is **7 days** if you create the
        DB instance using the console.

    4. Default backup retention period is **1 day** if you create the
        DB instance using the RDS API or the AWS CLI.

    5. Multi-AZ: Automated backups are taken from **standby**
        (experience latency for a **few mins** for Multi-AZ).

    6. Single-AZ: I/O activity is suspended on your **primary** during
        **backup**.

    7. **To** **retain backups longer than 35 days**, implement a
        **Lambda** function to initiate the **RDS snapshot**. The
        Lambda function can be triggered on a schedule using
        **CloudWatch Events Rule**.

    8. **To retain backups in a different region**, copy manual RDS DB
        snapshot to the secondary region.

        1. Although it is possible to configure RDS Read Replica with
            automated backups in the secondary region, this is not an
            optimal solution as it involves additional costs
            associated with the running RDS instance.

    1. RDS does not offer capability to copy automated backups (e.g. to
        S3).

    10. RDS does not offer capability to configure RDS automated backups
        to store data in a different region.

13. Manual Snapshots

    1. DB Snapshots are kept until you explicitly delete them.

    2. You cannot share a snapshot that has been encrypted using the
        default KMS encryption key.

    3. You cannot share encrypted snapshots as public.

    4. You cannot share Oracle or Microsoft SQL Server snapshots that
        are encrypted using Transparent Data Encryption (TDE).


14. To copy an encrypted snapshot from one Region to another, you must
    specify the **KMS key identifier** of the destination Region. This
    is because KMS CMKs are specific to the Region that they are
    created in.

    1. The source snapshot remains encrypted throughout the copy
        process. KMS uses envelope encryption to protect data during
        the copy process.

15. To share a **manual DB snapshot** with another AWS account
    ([Ref](https://docs.aws.amazon.com/en_pv/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html))

    1. Sharing a manual DB snapshot, whether encrypted or unencrypted,
        enables authorized AWS accounts to copy the snapshot.

    2. Sharing an unencrypted manual DB snapshot enables authorized AWS
        accounts to directly restore a DB instance from the snapshot
        instead of taking a copy of it and restoring from that.

    3. However, you cannot restore a DB instance from a **shared
        encrypted** DB snapshot. Instead, you can make a copy of the
        DB snapshot and restore the DB instance from the copy.

        1. You must also share the **KMS CMK** that was used to encrypt
            the snapshot with any accounts that you want to be able to
            access the snapshot (by adding the other account to the
            **KMS key policy**).

            1.  Principal: ARN of the AWS account (root) that you are
                sharing to

            2.  Allow: kms:CreateGrant

    4. You cannot share a DB snapshot that uses an **option group with
        permanent or persistent options**.

16. To share an **automated DB snapshot**, create a manual DB snapshot
    by copying the automated snapshot, and then share that copy.
    ([Ref](https://docs.aws.amazon.com/en_pv/AmazonRDS/latest/UserGuide/USER_ShareSnapshot.html))

17. Neither automated backups nor DB Snapshots can be taken from your
    Read Replicas.

18. The **database cloning** feature is only available in Aurora, not
    RDS.

19. Restoring from a DB snapshot
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html))

    1. You cannot restore a DB instance from a **shared** **encrypted**
        DB snapshot. Instead, you can make a copy of the DB snapshot
        and restore the DB instance from the copy.

    2. To restore a DB instance from a **shared snapshot** using the
        AWS CLI or API, the **snapshot ARN** must be used as the
        snapshot identifier.

    3. You can specify the **parameter group** when you restore the DB
        instance.

        1. You should retain the **parameter group** for any DB
            snapshots you create, so that you can associate your
            restored DB instance with the correct **parameter group**.

    4. When you restore a DB instance, the **option group** associated
        with the DB snapshot is associated with the restored DB
        instance after it is created.

    5. When restoring a DB instance to a specific PIT (automatic
        backup) or from a manual snapshot, the **default DB security
        group** is applied to the new DB instance.

        1. If you need custom DB security groups applied to your DB
            instance, you must apply them explicitly
            (**modify-db-instance**) after the DB instance is
            available.
            ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html))

        2. After a restore, if you experience consistent connection
            timeout errors in related logs following each refresh
            operation, it could be because you have the default
            security group associated with the restored.

20. The **PIT restore** and **snapshot restore** features of RDS require
    a crash-recoverable storage engine.

    1. **InnoDB** is the only supported storage engine for **RDS
        MySQL**.
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html))

        1. **InnoDB storage engine** supports automated backups on
            **RDS MySQL**.

        2. **InnoDB** instances can also be migrated to Aurora.

        3. **MyISAM** instances cannot be migrated to Aurora. However
             **MyISAM** performs better than **InnoDB** if you require
             intense, full-text search capability.

        4. **Federated** Storage Engine is not supported by RDS.

    2. **InnoDB** and **XtraDB** are the supported storage engines for
        **RDS MariaDB**.

        1. **Aria** storage engine is not supported by RDS.

    3. Recommend these limits because having large numbers of tables
        significantly increases database recovery time after a
        failover or database crash.

        1. max **10000** tables if **io1** or **gp2** (>= 200 GiB);

        2. max **1000** tables if **magnetic** or **gp2** (<200 GiB)

        3. If you need to create more tables than recommended, set the
             **innodb_file_per_table parameter to 0**.

    4. When attempting a point-in-time restore (PITR), the RDS MySQL DB
        instance status becomes "incompatible-restore":

        1. RDS cannot do a point-in-time restore due to the existence
            of temporary tables.

21. Tune the most used and most expensive queries to see if that lowers
    the pressure on system resources.

22. Enable automatic backups and set the backup window to occur during
    the daily low in write IOPS.

23. If you have scaled the compute and/or storage capacity of the source
    DB Instance, you should also scale the Read Replicas, which should
    have as much or more compute and storage resources as their
    respective source DB Instances for replication to work
    effectively.

24. If your database workload requires more I/O than you have
    provisioned, recovery after a failover or database failure will be
    slow. To increase the I/O capacity of a DB instance:

    1. Convert from standard storage to either **gp2** or **io2 /
        io1**.

    2. Migrate to a DB instance class with High I/O capacity.

    3. If you convert to Provisioned IOPS storage, make sure you also
        use a DB instance class that is optimized for Provisioned
        IOPS.

    4. If you are already using Provisioned IOPS storage, provision
        additional throughput capacity.

25. For best **IOPS** performance, allocate enough RAM so that your
    working set resides almost completely in memory

    1. The working set is the data and indexes that are frequently in
        use on your instance. The more you use the DB instance, the
        more the working set will grow.

    2. To tell if your working set is almost all in memory, check the
        **ReadIOPS** metric using CloudWatch while the DB instance is
        under load.

        1. The value of **ReadIOPS** should be small and stable.

        2. If scaling up the DB instance class (to a class with more
            RAM) results in a dramatic drop in ReadIOPS, your working
            set is not almost completely in memory.

        3. Continue to scale up until ReadIOPS no longer drops
             dramatically after a scaling operation, or ReadIOPS is
             reduced to a very small amount.

26. **RDS Proxy** is a service that can be used to pool simultaneous
    connections from serverless applications and alleviate the
    connection management from the RDS database instance.

27. The **best number of user connections** for a DB instance will vary
    based on the instance class and the complexity of the operations
    being performed.

    1. You can associate your DB instance with a **parameter group**
        where the **User Connections** parameter is set to other than
        0 (unlimited).

28. Investigate **disk space consumption** if space used is consistently
    at or above **85% of the total disk space**. See if it is possible
    to delete data from the instance or archive data to a different
    system to free up space.

29. **High CPU or RAM consumption** might be appropriate, provided that
    they are in keeping with your goals for your application (like
    throughput or concurrency) and are expected.

30. To develop a cost-effective disaster recovery plan that will restore
    the database in a different Region within 2 hours (RTO), and the
    restored database should not be missing more than 8 hours of
    transactions (RPO). (4)

    1. **Backup-and-restore** is the most cost-effective solution to
        provide a 2-hour RTO and 8-hour RPO.

    2. Schedule a Lambda function to create an hourly snapshot of the
        DB instance, and another Lambda function to copy the snapshot
        to the second Region.

        1. Taking the snapshots every hour will keep the incremental
            snapshot size low, reduce the time to copy the snapshot
            across Regions, and meet the RPO.

    3. For disaster recovery, create a new RDS Multi-AZ DB instance
        from the last snapshot.

31. Failing to connect to a newly created database could be because:

    1. The database instance state is not yet available.

    2. The instance must have a public ip address.

    3. The inbound rules on the instance Security Group are not
        configured properly.

32. If encountering a connection timed out error when attempting to
    query a RDS PostgreSQL DB using pgAdmin:

    1. The corporate firewall is blocking access to port 5432. Update
        the DB and SG settings to use a different port.

    2. The DB instance is not publicly accessible. Create an IGW for
        the subnets in the DB subnet group.

33. **Max 40** RDS DB instances by **default**. (**Max 10** for
    **Oracle** or **SQL Server** if under "License Included" model).

34. Unlimited number of databases and schemas can be run within a DB
    instance; except

    1. RDS for **SQL Server**: **max 100 databases per instance**

    2. RDS for **Oracle**: **1 database per instance**; no limit on
        number of schemas per database imposed by software

35. Deprecated DB engine version

    1. When a **minor version** of a DB engine is deprecated in RDS, at
        the end of a **3 month** period after the announcement, all
        instances still running the deprecated minor version will be
        scheduled for automatic upgrade to the **latest supported
        minor version** during their scheduled maintenance windows.

    2. When a **major version** of a DB engine is deprecated in RDS, at
        the end of a minimum **6 month** period after the
        announcement, an automatic upgrade to the **next major
        version** will be applied to any instances still running the
        deprecated version during their scheduled maintenance windows.

    3. Once a major or minor database engine version is no longer
        supported in RDS, any DB instance restored from a DB snapshot
        created with the unsupported version will automatically and
        immediately be upgraded to a currently supported version.

36. Hybrid or on-premises deployment options: [RDS on
    Outposts](https://aws.amazon.com/rds/outposts/faqs/) and
    [RDS on VMware](https://aws.amazon.com/rds/vmware/faqs/).


---
## Parameter Groups, DB Cluster Parameter Groups, Option Groups

37. A **DB parameter group** contains engine configuration values that
    are applied to one or more DB instances.
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_WorkingWithParamGroups.html))

    1. To limit the no. of simultaneous connections that a user account
        can make on a RDS MySQL DB instance:

        1. Create a new custom **DB parameter group**.

        2. Modify the **max_user_connections** parameter to 10.

        3. Update the RDS MySQL DB instance to use the new **parameter
             group**.

38. A **DB cluster parameter group** contains engine configuration
    values that are applied to all DB instances in a **DB cluster**.

    1. The Aurora shared storage model requires that every DB instance
        in an Aurora cluster use the same setting for parameters such
        as **innodb_file_per_table**.

    2. To enable **audit logs** of a Aurora MySQL DB cluster, including
        database events e.g. connections, disconnections, tables
        queried, or types of queries issued (DML, DDL, or DCL).

        1. Create a **DB cluster parameter group**, and configure the
            **Advanced Auditing** parameters. And then associate the
            **custom parameter group** with the Aurora DB cluster.

        2. Modify the **log export configuration** of the RDS cluster
            to publish logs to CloudWatch (log group).

39. A custom **option group** is used to enable and configure additional
    features provided by **some DB engines**.
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithOptionGroups.html))

    1. To enable MySQL (or MariaDB) **audit logs** using
        **MARIADB_AUDIT_PLUGIN**.

    2. To perform SQL Server native backup using .bak files (SQL Server
        specific functionality).

        1. Create a new custom **option group** and configure the
            **SQLSERVER_BACKUP_RESTORE** setting, and then associate
            the new **option group** with the RDS SQL Server DB
            instance.

40. You cannot modify the parameter settings of a **default parameter
    group**. Instead, you create your own **custom parameter group**
    where you choose your own parameter settings.

41. Not all DB engine parameters can be changed in a **parameter group**
    that you create.

42. **Static parameters** require a manual reboot of the RDS instance
    before they are applied.

43. **Dynamic parameters** do not require a manual reboot and are
    applied immediately regardless of the Apply Immediately setting.

44. You must reboot the instance for the changes to take effect, if you
    ([Ref](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RebootInstance.html))

    1. modify a DB instance,

    2. change the **DB parameter group** associated with the instance,
        or

    3. change a **static DB parameter** in a parameter group the
        instances use.

45. Troubleshooting

    1. A RDS MySQL database instance is failing to reboot. Event logs
        show an error: "MySQL could not be started due to
        **incompatible parameters**".

        1. Compare the RDS DB instance **DB parameter group** to the
            default parameter.

        2. Reset any **custom parameters** to their default value.

        3. Reboot the instance.

        4. Because one (or more) parameters are set to non-default
            values that are not compatible with the current RDS engine
            or instance class. To resolve the issue, you must reset
            the parameters to their default values.

        5.  You cannot modify the RDS instance that is in an
            **incompatible parameter state**. You must reset the
            values of the DB parameter group currently applied to the
            RDS instance.

        6.  RDS does not allow modification of MySQL system variables
            directly using the SET statement. Instead DB parameter
            groups must be used.

    2. The time zone of a **RDS MariaDB instance** has been updated by
        setting the dynamic parameter **time_zone** in the DB
        parameter group to the local time zone of the application. But
        an application user is still reporting an incorrect time zone.

        1. Ensure that the DB Parameter Group is applied to the RDS
            instance.

        2. Instruct the application user to disconnect from the
            database and start a new session.

        3. Dynamic parameters in DB parameter groups do not require
             RDS instance reboot.

        4. **rdsadmin.rdsadmin_util.alter_db_time_zone** procedure is
            used to set the time zone of an **Oracle** DB instance.

---
## Monitoring, Logging and Alerting

46. MySQL / MariaDB

    1. **MySQL error log** contains diagnostic messages generated by
        the database engine along with startup/shutdown times. Enabled
        by default. mysql-error.log is flushed **every 5 mins**;
        appended also to mysql-error-running.log.

    2. **MySQL audit log** records database activity on the server for
        audit purposes, must be enabled using **Option group** with
        the **MARIADB_AUDIT_PLUGIN** option.

    3. **MySQL slow query and general logs** must be enabled using **DB
        Parameter groups**
        **(**[Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Concepts.MySQL.html)**)**:

        1. **general_log**: contains a record of all SQL statements
            received from clients, and client connect and disconnect
            times. (0 or 1, default 0)

        2. **slow_query_log**

        3. **long_query_time**: To prevent fast-running queries from
             being logged in the slow query log, default 10s

        4. **log_queries_not_using_indexes**

        5.  **log_output** (optional)

            1.  **TABLE** (default) to **mysql.general_log** table and
                **mysql.slow_log** table.

            2.  **FILE** to write both logs to the file system and
                publish them to CloudWatch Logs.

47. PostgreSQL

    1. **PostgreSQL error log** contains connections/disconnections,
        checkpoints, autovacuum information and rds_admin actions in
        the error log.

    2. **PostgreSQL query log** must be enabled using **DB Parameter
        groups**

        1. Set the **log_statement** parameter to **all**.

        2. Set the **log_min_duration_statement** parameter. Write to
            the postgres.log file when set to **1**.

48. **RDS Performance Insights** displays DB load and the top SQL
    regardless of engine.

    1. Performance Insights works best when the **MySQL Performance
        Schema** is enabled.

    2. For Aurora PostgreSQL, DB load is subdivided by PostgreSQL 10
        wait events.

    3. For Aurora MySQL and RDS MySQL, DB load is subdivided by MySQL
        Performance Schema wait events.

49. **RDS Event Notification** can be enabled and provides notifications
    for various categories of database events.
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_Events.html))

    1. RDS uses SNS to provide a notification when an RDS event occurs.

    2. These events can be configured for source categories: DB
        instance, DB security group, DB snapshot and DB parameter
        group. (NOT DB option group).

    3. To set up a notification if the automatic backups are ever
        turned off, subscribe to RDS Event Notification and be sure to
        include the **configuration change** event "Automatic backups
        for this DB instance have been disabled".

    4. E.g. Configuration change event: "The master password for the DB
        instance has been reset".

50. **CloudWatch Application Insights** for databases collects
    performance metrics and helps in troubleshooting by automatically
    correlating errors and creating visual dashboards.
    ([Ref](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/appinsights-what-is.html#appinsights-stack))

    1. DynamoDB table

    2. MySQL running on RDS, Aurora, or EC2

    3. PostgreSQL running on RDS or EC2

    4. Microsoft SQL Server running on RDS or EC2

    5. Oracle running on RDS or EC2

51. You have a highly available production 10 TB SQL Server running on
    EC2. How to provide metrics visibility and notifications to
    troubleshoot performance and connectivity issues?

    1. Configure CloudWatch Application Insights for .NET and SQL
        Server to monitor and detect signs of potential problems.
        Configure CloudWatch Events to send notifications to an SNS
        topic.

52. **CloudWatch** collects performance metrics of EC2 instances, not
    RDS instances. Further it collects host level metrics, and cannot
    identify issues relating to individual SQL queries.

---
## Storage volume of managed services: Aurora, DocumentDB, Neptune

53. Managed services (Aurora, DocumentDB, Neptune) use a distributed
    storage volume that is shared across the database instances. The
    auto-healing storage **volume automatically replicates data 6
    times across 3 AZs** for durability and availability. The **volume
    automatically scales** with your dataset, up to **64 TiB**.

    See [Image](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.html)


---
## Aurora (Single-Master Clusters)


54. **Aurora Singe-Master Clusters** (include **provisioned**,
    **parallel query**, **global database**, **serverless**)

    1. Parallel Query is not compatible with Serverless and Backtrack
        features.

    2. Backtracking is NOT supported with binary log replication
        (MySQL). **Cross-region replication** must be **disabled**
        before you can configure or use Backtracking.

    3. MySQL vs. PostgreSQL

        1. Features supports Aurora MySQL but not Aurora PostgreSQL
            ([FAQs](https://aws.amazon.com/rds/aurora/faqs/))

            1.  Multi-Master clusters

            2.  Parallel query

            3.  Backtrack

            4.  **Read replica write forwarding** feature in Aurora
                global database

        2. Aurora MySQL Max table size (= Max Cluster Volume) = **128
            TiB**

        3. Aurora PostgreSQL Max table size = **32 TiB**

55. Interface: **SQL**

56. Availability, reliability, fault tolerance, upgrade

    1. Aurora automatically promotes a **Replica** (in the same or
        different AZ) to become the new primary and flips the CNAME,
        **complete within 30 seconds**.

    2. If there's no replica (i.e. single instance), Aurora will
        attempt to create a new DB Instance in the same AZ on a
        best-effort basis and may not succeed, and will take longer
        time.

    3. **Global Database** can promote a secondary region to take full
        read/write workloads **in < 1 min**
        ([Ref](https://aws.amazon.com/rds/aurora/faqs/)).

    4. Instance scaling will have an availability impact for **a few
        minutes**.

    5. DB engine maintenance (optional **auto minor version upgrade**,
        manual major engine version upgrade).
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Updates.html))

        1. Database engine version upgrades are applied to all
            instances in a DB cluster at the same time.

        2. An update requires a database restart on all instances in a
            DB cluster, (**20 - 30 seconds of downtime)**, after which
            you can resume using your DB cluster.

57. Multi-AZ and automatic failover
    ([optional](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html))

    1. When data is written to the primary DB instance, Aurora
        **synchronously** replicates the data across multiple AZs to
        **6 storage nodes** associated with your cluster volume in a
        single Region, regardless of whether the instances in the DB
        cluster span multiple AZs.

    2. When the primary DB instance becomes unavailable, Aurora
        automatically promotes a **Replica** (in the same or different
        AZ) to become the new primary and flips the CNAME, **complete
        within 30 seconds**.

    3. You can specify the **failover priority** for Aurora Replicas.

        1. Priorities range from 0 for the first priority to 15 for the
            last priority.

        2. If more than one Aurora replicas have the same priority, RDS
            promotes the replica that is largest in size.

    4. To ensure fast failover with Aurora PostgreSQL
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html))

        1. Use the provided read and write Aurora endpoints to
            establish a connection to the cluster.

        2. Aggressively set **TCP keepalives** (e.g. 1s) to ensure that
            longer running queries that are waiting for a server
            response will be killed before the read timeout expires in
            the event of a failure.

        3. Set the **Java DNS caching** timeouts aggressively (e.g.
             1s) to ensure the Aurora read-only endpoint can properly
             cycle through read-only nodes on subsequent connection
             attempts.

        4. Set the **timeout variables** used in the **JDBC
            connection** string as **low** as possible. Use **separate
            connection objects** for **short and long** running
            queries.

        5.  Use **RDS APIs** to test application response on server side
            failures and use a **packet dropping tool** to test
            application response for client-side failures.

    5. Minimal application downtime during a failover

        1. Enable **Aurora DB cluster cache management**.

        2. Set a low value for the database and application client
            **TCP keepalive** parameters.

    6. To initiate a failover for testing purpose from API
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html))

        1. FailOverDBCluster

        2. ModifyDBInstance

        3. DeleteDBInstance

    7. The most operationally efficient approach for testing failover
        capabilities of a Aurora MySQL DB cluster in a single AZ.

        1. Add a new reader instance with a different AZ to the cluster
            to make it a multi-AZ deployment. Simulate an instance
            crash using the **ALTER SYSTEM CRASH** fault injection
            query and run the **failover-db-cluster** CLI command for
            the DB to failover.

58. Multi-region = cross-region read replicas
    ([optional](https://aws.amazon.com/rds/aurora/faqs/)):

    1. Physical replication: Aurora Global Database

    2. Native logical replication: You can replicate to Aurora and
        non-Aurora databases, across regions (binlog for MySQL, and
        PostgreSQL replication slots for PostgreSQL)

    3. Logical replication: Aurora MySQL also offers an easy-to-use
        logical cross-region read replica.

59. **Aurora single-master cluster** consists of:
    ([UserGuide](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.html),
    [FAQs](https://aws.amazon.com/rds/aurora/faqs/))

    1. One **primary DB instance**, supports read/write operations,
        performs all data modifications to the **cluster volume**.

    2. One **cluster volume**, a SSD-backed virtual database storage
        volume that spans multiple AZs, with each AZ having a copy of
        the DB cluster data.

    3. Max **15 Read Replicas**, support only read operations, connect
        to the same storage volume as the primary DB instance.
        Maintain high availability by locating Aurora Read Replicas in
        separate AZs.

60. Endpoints of a Aurora cluster
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html))

    1. One **cluster endpoint** (or **writer endpoint**), connects to
        the current primary DB instance for the DB cluster.

        1. Example endpoint:
            mydbcluster.cluster-123456789012.us-east-1.rds.amazonaws.com:3306

        2. The cluster endpoint can be used for read and write
            operations.

        3. You use the cluster endpoint for all write operations on
             the DB cluster, including inserts, updates, deletes, and
             DDL (data definition language) and DML (data manipulation
             language) changes.

        4. The cluster endpoint is the one that you connect to when you
            first set up a cluster or when your cluster only contains
            a single DB instance.

    2. One **reader endpoint** provides **automatic load-balancing**
        support for read-only connections to the DB cluster.

        1. Example endpoint:
            mydbcluster.cluster-ro-123456789012.us-east-1.rds.amazonaws.com:3306

        2. If the cluster only contains a primary instance and no read
            replicas, the reader endpoint connects to the primary
            instance. In that case, you can perform write operations
            through the endpoint.

        3. If the cluster contains one or more Replicas, the reader
             endpoint load-balances each connection request (not read
             requests) among the Replicas. In that case, you can only
             perform read-only statements such as SELECT in that
             session.

    3. **0 - 5** **custom endpoints** for an Aurora cluster represents
        a set of DB instances that you choose.

        1. Example endpoint:
            myendpoint.cluster-custom-123456789012.us-east-1.rds.amazonaws.com:3306

        2. When you connect to the endpoint, Aurora performs load
            balancing and chooses one of the instances in the group to
            handle the connection, based on criteria other than the
            read or read/write capability of the DB instances; e.g.
            based on instance class or DB parameter group

            1.  E.g. You might direct internal users to low-capacity
                instances for report generation or ad hoc (one-time)
                querying, and direct production traffic to
                high-capacity instances.

        3. You cannot use **custom endpoints** for **Aurora
             Serverless** clusters.

    4. **Instance endpoint** maps directly to a cluster instance

        1. Each and every cluster instance has its own instance
            endpoint.

61. Scaling - Storage
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Performance.html))
    ([5 Scaling categories](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Performance.html))

    1. Aurora storage automatically scales with the data in your
        cluster volume, up to **128 TiB (tebibytes)**.

    2. The storage space allocated to your Aurora database cluster
        dynamically decreases when you delete data from the cluster
        (since 2020-10).

    3. For earlier versions without the dynamic resizing feature,
        resetting the storage usage for a cluster involved doing a
        logical dump and restoring to a new cluster. That operation
        can take a long time for a substantial volume of data. If you
        encounter this situation, consider upgrading your cluster to a
        version that supports volume shrinking.

    4. When you **clone** an RDS DB instance, the **storage
        autoscaling** setting is not automatically inherited by the
        cloned instance. The new DB instance has the same amount of
        allocated storage as the original instance.

        1. The **database cloning** feature is only available in
            Aurora, not RDS.

62. Scaling - Instance Scaling
    ([Ref](https://aws.amazon.com/rds/aurora/faqs/))

    1. A change on DB Instance class will be applied during the
        specified maintenance window; use the "Apply Immediately"
        flag to apply your scaling requests immediately.

    2. Instance scaling will have an availability impact for **a few
        minutes** as the scaling operation is performed.

    3. Any other pending system changes will also be applied.

    4. Aurora PostgreSQL does not support **Multi-Master Cluster** -
        i.e. can only have one master doing all writes. You will need
        to use vertical scaling (or instance scaling) approach for
        improving write operation when needed.

63. Scaling - Read Scaling

    1. Max **15** **read-only Aurora Replicas** in a DB cluster that
        uses single-master replication.

    2. Scaling for Write operations only supported by **Aurora
        Multi-Master cluster.**

    3. **Second-tier read replica**
        ([Ref](https://aws.amazon.com/rds/faqs/))

        1. Supported (Aurora, RDS MySQL/MariaDB); not supported (RDS
            PostgreSQL/Oracle/SQL Server).

        2. You can create a second-tier Read Replica from an existing
            first-tier Read Replica. By creating a second-tier Read
            Replica, you may be able to move some of the replication
            load from the master database instance to a first-tier
            Read Replica.

        3. A second-tier Read Replica may lag further behind the
             master because of additional replication latency
             introduced as transactions are replicated from the master
             to the first tier replica and then to the second-tier
             replica.

64. Scaling - Managing Connections

    1. The maximum number of connections allowed is determined by the
        **max_connections** parameter in the **instance-level
        parameter group** for the DB instance.

    2. The default value of that parameter varies depending on the DB
        instance class used for the DB instance and database engine
        compatibility.

65. Scaling - Managing Query Execution Plan.

    1. If you use query plan management for Aurora PostgreSQL, you gain
        control over which plans the optimizer runs.

66. Cross-region read replicas
    ([Ref)](https://aws.amazon.com/rds/aurora/faqs/)

    1. **Asynchronous** replication

    2. All regions are accessible and can be used for **reads**.

    3. Database engine version upgrades happen on **all instances**
        together.

    4. Automated backups can be taken in **each region**.

    5. Each region can have a Multi-AZ deployment.

    6. Aurora allows promotion of a secondary region to be the master.

    7. **(1) Aurora Global Database** (physical replication)
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html))

        1. Global Database clusters are **single-master clusters**
            providing subsecond replication across Regions.

        2. Only the primary cluster performs write operations. Clients
            that perform write operations connect to the **DB cluster
            endpoint** of the primary cluster.

        3. A secondary cluster does not have a writer primary DB
             instance. This functionality means that it can have up to
             **16 replica instances**, instead of the limit of 15 for
             a single Aurora cluster.

        4. Global Database can replicate to up to **5 secondary
            regions** with typical **latency < 1 sec**.

        5.  Global Database can promote a secondary region to take full
            read/write workloads in **< 1 min**.

        6.  **For low-latency global reads** (not write) and **disaster
            recovery.**

        7.  Global Database uses a primary instance for write
             operations. Although there is automatic fail-over
             capacity, it is **not without any downtime**.

        v3. Global Database's write operations are issued directly to
              the primary DB instance in the primary Region. This does
              **not reduce latency of write operations**.

        ix. **Enable read replica write forwarding** feature of **Aurora
            MySQL** (not PostgreSQL)
            ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-global-database-write-forwarding.html))

            1.  You can reduce the number of endpoints that you need to
                manage for applications running on your Aurora global
                database, by using **write forwarding**.

            2.  Secondary clusters in an Aurora global database forward
                SQL statements that perform write operations to the
                primary cluster. The primary cluster updates the
                source and then propagates resulting changes back to
                all secondary Regions.

    8. **(2) Native logical replication** to Aurora and non-Aurora
        databases, even across regions.
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.MySQL.html))

        1. PostgreSQL: **PostgreSQL replication slots**

        2. MySQL: **binlog** (binary logging)

        3. You can do replication (or cross-region replication) using
             binary log replication **from/to** Aurora MySQL, RDS
             MySQL, external SQL.

             1.  Enable binary logging on the replication source.

             2.  Retain binary logs on the replication source until no
                 longer needed.

             3.  Create a snapshot of your replication source

                 1. However, if the replication source is Aurora DB
                     cluster snapshot but the replica target is one of
                     the follow:

                     1. an Aurora DB cluster owned by another AWS
                         account,

                     2. an external MySQL database, or

                     3. an RDS MySQL DB instance,

                 2. Instead, create a dump of your Aurora DB cluster by
                     connecting to your DB cluster using a **MySQL
                     client** and issuing the **mysqldump** command.

             4.  Load the snapshot into your replica target

             5.  Enable replication on your replica target

             6.  Monitor your replica

        4. Example: To replicate data in an Aurora MySQL database
            cluster to a RDS MySQL database instance in another
            Region: (3)

            1.  Enable binary logging on the development Aurora
                database. Retain binary logs until no longer needed.

            2.  Load the snapshot into the RDS MySQL instance by copying
                the output of the **mysqldump** command from your
                **replication master (source)**.

    1. **(3) Aurora MySQL also offers an easy-to-use logical
        cross-region read replica**
        ([Ref](https://aws.amazon.com/rds/aurora/faqs/))

        1. Support up to **5 secondary Regions**.

        2. It is based on single threaded **binlog replication**, so
            the replication lag will be influenced by the change/apply
            rate and delays in network communication between the
            specific regions selected.

        3. To enable **binary logging** (to replay changes on the
             cross-Region read replica DB cluster)
             ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.CrossRegion.html#AuroraMySQL.Replication.CrossRegion.Prerequisites))

             1.  Update the **binlog_format** parameter for the source
                 Aurora DB cluster.

             2.  If your DB cluster uses the default DB **cluster
                 parameter group**, create a new DB **cluster
                 parameter group** to modify **binlog_format**
                 settings.

             3.  We recommend that you set the binlog_format to
                 **MIXED**. However, you can also set binlog_format to
                 **ROW** or **STATEMENT** if you need a specific
                 binlog format.

             4.  Reboot your Aurora DB cluster for the change to take
                 effect.

             5.  Create a cross region read replica in the target
                 region.

        4. When you're performing replication across Regions, ensure
            that your DB clusters and DB instances are publicly
            accessible. Aurora MySQL DB clusters must be part of a
            public subnet in your VPC.
            ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.MySQL.html))

        5.  It is possible to set up replication where your Aurora MySQL
            database cluster is the replication master (MySQL
            feature), but this is not mandatory. It could also be set
            up as the replica.

67. When using Aurora MySQL to store transactional data, what can cause
    a replication failure?
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/CHAP_Troubleshooting.html#CHAP_Troubleshooting.MySQL))

    1. Binary logging on the primary database is not enabled.

    2. The maximum allowed packets for the read replica does not equal
        the primary database maximum.

68. Deleting a Read Replica DB cluster

    1. It is impossible to delete the last instance of a Read Replica
        DB cluster. A Read Replica cluster must be promoted to a
        standalone database cluster before it can be deleted.

    2. **Deletion protection** on the source database does not affect
        deletion protection on read replicas.

    3. You do not need to shut down a read replica in order to delete
        it.

69. **Backtracking** an Aurora DB cluster (**support MySQL, not
    PostgreSQL**)
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html))

    1. Backtrack rewinds a Aurora MySQL DB cluster to a specific time
        **in minutes**, without creating a new cluster.

    2. The **target backtrack window** can go as far back as **72
        hours**.

    3. The **actual backtrack window** is based on your workload and
        the storage available for storing information about database
        changes, called **change records**.

    4. Aurora retains **change records** for the **target backtrack
        window**, and you pay an hourly rate for storing them.

    5. You can enable the Backtrack feature when you create a new DB
        cluster or restore a snapshot of a DB cluster.

    6. Backtracking causes a brief DB instance disruption. You must
        **stop or pause your applications** before starting a
        backtrack operation to ensure that there are no new read or
        write requests.

        1. During the backtrack operation, Aurora pauses the database,
            closes any open connections, and drops any uncommitted
            reads and writes. It then waits for the backtrack
            operation to complete.

    7. Backtracking is NOT supported with binary log (binlog)
        replication. **Cross-region replication** must be **disabled**
        before you can configure or use backtracking.

    8. You can NOT use Backtrack with Aurora multi-master clusters.

    1. A developer is doing some testing on an application using Aurora
        database. During the testing activities, the developer
        accidentally executes a DELETE statement without a WHERE
        clause. They wish to undo this action.

        1. Use **Aurora Backtracking** feature, which enables one to
            revert an Aurora cluster to a specific point in time,
            without restoring data from a backup.

        2. "Restore to point in time" feature, will launch a new
            cluster and restore it from backup data. Restoring from
            backup is a very time-consuming process that can take
            hours to complete. Further, this solution has costs
            associated with the additional Aurora cluster.

        3. "Restore the Aurora database from a snapshot" does not
             restore data to a specific desired point in time.
             Additionally, restoring from a snapshot launches a new
             cluster, thus incurring additional costs. Since the
             restore happens from backup data, it can take hours to
             complete. Therefore, this is not the optimal solution.

        4. Restore the Aurora from a read replica. - it is not possible
            to perform a restore from a read replica.

70. **Database Cloning** in an Aurora DB Cluster can quickly and
    cost-effectively create clones of all of the databases within an
    Aurora DB cluster.
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Clone.html))

    1. The clone databases require only minimal additional space when
        first created.

    2. Database cloning uses a [**[copy-on-write
        protocol]**](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Clone.html#Aurora.Managing.Clone.Protocol),

        1. Initially, both the source database and the clone database
            point to the same pages. None of the pages has been
            physically copied, so no additional storage is required.

        2. When the **source database** makes a change to the data
            (e.g. in page 1), instead of writing to the original page
            1, a new page (e.g. page 1') is created and the source
            database now points to the new page 1'. The **clone
            database** continues to point to Page 1.

        3. When the **clone database** makes a change to the data
             (e.g. in page 4), instead of writing to the original page
             4, a new page (e.g. page 4') is created and the clone
             database now points to the new page 4'. The **source
             database** continues to point to Page 4.

    3. You can make multiple clones from the same DB cluster.

    4. You can also create additional clones from other clones.

    5. You cannot create clone databases across AWS regions. The clone
        databases must be created in the same region as the source
        databases.

    6. Max **15 clones** based on a copy, including clones based on
        other clones. After that, only copies can be created. However,
        each copy can also have up to **15 clones**.

    7. You cannot clone from a cluster without the **parallel query**
        feature, to a cluster where **parallel query** is enabled.

        1. To bring data into a cluster that uses parallel query,
            create a snapshot of the original cluster and restore it
            to a cluster where the parallel query option is enabled.

    8. You can provide a different VPC for your clone. But, the subnets
        in those VPCs must map to the same set of AZ.

    1. When deleting a source database that has one or more clone
        databases associated with it, the clone databases are not
        affected. The clone databases continue to point to the pages
        that were previously owned by the source database.

    10. Limitations of **cross-account cloning**:

        1. The maximum number of cross-account clones that you can have
            for any Aurora cluster is **15**.

        2. You cannot clone an Aurora Serverless cluster across AWS
            accounts.

        3. You cannot clone an Aurora global database cluster across
             AWS accounts.

        4. Viewing and accepting sharing invitations requires using the
            AWS CLI the RDS API, or the AWS RAM console. Currently,
            you cannot perform this procedure using the RDS console.

        5.  When you make a cross-account cluster, you cannot make
            additional clones of that new cluster or share the cloned
            cluster with other AWS accounts.

        6.  Your cluster must be in ACTIVE state at the time that you
            share it with other AWS accounts.

        7.  While an Aurora cluster is shared with other AWS accounts,
             you cannot rename the cluster.

        v3. You cannot create a cross-account clone of a cluster that
              is encrypted with the default RDS key.

        ix. When an encrypted cluster is shared with you, you must
            encrypt the cloned cluster. The key you use can be
            different from the encryption key for the original
            cluster. The cluster owner must also grant you permission
            to access the KMS key for the original cluster.

    11. Use cases

        1. Experiment with and assess the impact of changes, such as
            schema changes or parameter group changes.

        2. Perform workload-intensive operations, such as exporting
            data or running analytical queries.

        3. Create a copy of a production DB cluster in a nonproduction
             environment for development or testing.

71. Encryption: optional

<.br>Enabling encryption for RDS and Aurora with minimal downtime

    1. For an Aurora encrypted DB cluster, all DB instances, logs, backups,
        and snapshots are encrypted.

    2. You cannot disable encryption on an encrypted DB cluster.

    3. You cannot create an encrypted snapshot of an unencrypted DB
        cluster.

    4. A snapshot of an encrypted DB cluster must be encrypted using the
        same CMK as the DB cluster.

    5. You cannot convert an unencrypted DB cluster to an encrypted one.
        However, you can restore an **unencrypted snapshot** to an
        encrypted Aurora DB cluster. To do this, specify a CMK when you
        restore from the unencrypted snapshot.

    6. You cannot unencrypt an encrypted DB cluster. However, you can
        export data from an encrypted DB cluster and import the data into
        an unencrypted DB cluster.

    7. You cannot create an encrypted Aurora Replica from an unencrypted
        Aurora DB cluster. You cannot create an unencrypted **Aurora
        Replica** from an encrypted Aurora DB cluster.

    8. KMS supports CloudTrail, so you can audit CMK usage to verify that
        CMKs are being used appropriately.

    1. If an instance status becomes inaccessible-encryption-credentials

        1. The encryption key that the Aurora cluster uses may have been
            disabled. The Aurora cluster is no longer available, and the
            current state of the database cannot be recovered. The DBA
            must re-enable the key and restore the cluster.

    10. Manipulating the encrypted database snapshots
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Overview.Encryption.html))

        10. To copy an encrypted snapshot from one Region to another, you must
            specify the CMK in the destination Region. This is because CMKs
            are specific to the Region that they are created in.

        11. The source snapshot remains encrypted throughout the copy process.
            Aurora uses envelope encryption to protect data during the copy
            process.

72. Backup and Restore

    1. In the unlikely event your data is unavailable within Aurora
        storage, you can restore from a DB Snapshot or perform a PIT
        restore operation to a new instance.

    2. Backup retention period: **1 - 35 days** ([always
        on](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html))

    3. Aurora always has automated backups enabled on DB Instances
        (which allows PIT restore).

    4. Automated backups are taken from the **shared storage layer**.

    5. The latest restorable time for a PIT restore operation (5 mins
        in the past).

    6. Backups do not impact database performance.

    7. There is no performance impact when taking snapshots.

    8. It is not possible to perform a restore from a read replica.

73. **Database Activity Streams** (Aurora MySQL) provides a near
    real-time stream of database activities in your relational DB.

    1. Solutions built on top of **Database Activity Streams** can
        protect your database from internal and external threats.

    2. The collection, transmission, storage, and processing of
        database activity is managed outside your database, providing
        access control independent of your database users and admins.

    3. Your database activity is **asynchronously** pushed to an
        encrypted **Kinesis data stream** provisioned on behalf of
        your Aurora cluster.

Performance tuning for Aurora

Design or integration with other AWS services

74. Aurora MySQL is used to store user data. A team would like to
    enhance the application by sending a welcome email every time a
    new user is created. They've implemented an Lambda function that
    will generate email text and send it using SES.

    1. Create a trigger and stored procedure in the Aurora database. In
        the stored procedure, call **mysql.lambda_async** function to
        trigger the Lambda function.
        ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Lambda.html))

75. An Aurora MySQL has the message "Too many connections" in the error
    logs. How can it be resolved?

    1. Use RDS proxy to establish a connection pool between the
        application and the database.

    2. Scale the database up to an instance class with more memory.

---
## Aurora (Multi-Master Clusters)

76. **Aurora Multi-Master cluster**: cross multi-AZ, not multi-region
    ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-multi-master.html))

    1. **Aurora Multi-Master cluster** supports a **maximum of 4** DB
        instances in a multi-master cluster.

    2. The DB instances all have read/write capability and use the same
        AWS instance class.

    3. Multi-master clusters don't have a primary instance or
        read-only Aurora Replicas.

    4. There isn't any failover when a writer DB instance becomes
        unavailable, because another writer DB instance is immediately
        available to take over the work of the failed instance.

        1. We refer to this type of availability as **continuous
            availability**, to distinguish it from the **high
            availability** (with brief downtime during failover)
            offered by a single-master cluster.

    5. All the DB instances can write to the shared storage volume. For
        every data page you modify, Aurora automatically distributes
        several copies across multiple AZs.

    6. In applications where zero downtime is required for database
        write operations, a multi-master cluster can be used to avoid
        an outage when a **writer** instance becomes unavailable..

    7. All DB instances in a multi-master cluster must be in the **same
        Region** (not multi-region).

    8. You cannot enable cross-Region replicas from multi-master
        clusters.

    1. The **Stop action** isn't available for multi-master clusters.

    10. A multi-master cluster does not do any load balancing for
        connections. Your application must implement its own
        connection management logic to distribute read and write
        operations among multiple DB instance endpoints.

    11. You cannot take a snapshot created on a single-master cluster
        and restore it on a multi-master cluster, or the opposite.
        Instead, to transfer all data from one kind of cluster to the
        other, use a logical dump produced by a tool such as AWS
        Database Migration Service (AWS DMS) or the mysqldump command.

    12. You cannot use the parallel query, Aurora Serverless, or Global
        Database features on a multi-master cluster.

    13. The Performance Insights feature isn't available for
        multi-master clusters.

    14. You cannot clone a multi-master cluster.

    15. You cannot enable the backtrack feature for multi-master
        clusters.

---
## Aurora (Serverless Clusters V1)

77. Single AZ and automatic Multi-AZ failover ([Best
    practices)](https://aws.amazon.com/blogs/database/best-practices-for-working-with-amazon-aurora-serverless/)

    1. The DB instance for an Aurora Serverless DB cluster is created
        in a single AZ.

    2. The storage volume for the cluster is spread across multiple
        AZs. The durability of the data remains unaffected even if
        outages affect the DB instance or the associated AZ.

    3. Aurora Serverless clusters are monitored for availability and
        automatically replaced when problems are detected. Aurora
        Serverless manages the warmpool of pre-created DB clusters.
        The replacement process then fetches a new DB instance from
        the warmpooling service and replaces the unhealthy host.

    4. **Automatic Multi-AZ failover**

        1. In the unlikely event that an entire AZ becomes unavailable,
            Aurora Serverless launches a new instance for your cluster
            in one of the other AZs.

        2. This failover mechanism takes longer for Aurora Serverless
            than for an Aurora provisioned cluster.

        3. The Aurora Serverless AZ failover is done on a best effort
             basis because it depends on demand and capacity
             availability in other AZs within the given Region.

        4. Because of that Aurora Serverless is not supported by the
            Aurora Multi-AZ SLA.

78. Scaling

    1. Aurora Serverless uses a warm pool of servers to automatically
        scale capacity as a surge of activity is detected. It also
        scales back down automatically when the surge of activity is
        over.

    2. If autoscaling times out with finding a scaling point, by
        default Aurora keeps the current capacity. You can choose to
        have Aurora force the change by enabling the **Force the
        capacity change** option.

79. Pause and resume DB cluster

    1. You can choose to pause your Aurora Serverless DB cluster after
        a given amount of time with no activity. If enabled, default
        inactivity time is **5 minutes**.

    2. When the DB cluster is paused, no compute or memory activity
        occurs, and you are charged only for storage.

    3. If database connections are requested when an Aurora Serverless
        DB cluster is paused, the DB cluster automatically resumes and
        services the connection requests.

    4. When the DB cluster resumes activity, it has the same capacity
        as it had when Aurora paused the cluster. The number of ACUs
        depends on how much Aurora scaled the cluster up or down
        before pausing it.

    5. If a DB cluster is paused for more than **7 days**, the DB
        cluster might be backed up with a snapshot. In this case,
        Aurora restores the DB cluster from the snapshot when there is
        a request to connect to it.

80. DB cluster parameter group

    1. Unlike provisioned Aurora DB clusters, an Aurora Serverless DB
        cluster has a single read/write DB instance that is configured
        with a **DB cluster parameter group only.**

    2. does not have a separate DB parameter group.

    3. During autoscaling, Aurora Serverless needs to be able to change
        parameters for the cluster to work best for the increased or
        decreased capacity.

    4. Aurora Serverless PostgreSQL cannot use
        **apg_plan_mgmt.capture_plan_baselines** and other parameters
        that might be used on provisioned Aurora PostgreSQL DB
        clusters for **query plan management**.

81. Accessibility

    1. An Aurora Serverless DB cluster cannot have a public IP address.

    2. An Aurora Serverless DB cluster can be accessed from within a
        VPC or from a **Data API** HTTP endpoint.

    3. To connect to a Serverless cluster from a database client (e.g.
        from a Cloud9 env) inside the same VPC
        ([Ref](https://aws.amazon.com/getting-started/hands-on/configure-connect-serverless-mysql-database-aurora/)):

        1. Modify the cluster's VPC security group to allow network
            traffic from your DB client to access the cluster.

        2. Add a new Inbound Rule:

            MySQL/Aurora (3306), TCP (6), Port range (3306), Source: SG of Cloud9
            env

        3. From Cloud9 env:

                **mysql --user=(your Master username) --password -h (your
                database endpoint)**

        4. To connect to a Serverless cluster from a Lambda function, connect
            the Lambda function to the VPC via an ENI.
            ([Ref](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html))

            1. When you connect a Lambda function to a VPC, Lambda creates an
                ENI for each combination of SG and subnet in your function's
                VPC configuration.

            2. Multiple functions connected to the same subnets share ENIs, so
                connecting additional functions to a subnet that already has a
                Lambda-managed network interface is much quicker. However,
                Lambda might create additional network interfaces if you have
                many functions or very busy functions.

            3. If your function needs internet access, use NAT.

        5. To integrate a SaaS product with an Aurora Serverless DB cluster,
            the simplest and most secure solution is to enable **[Data
            API](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html)**
            on the Aurora Serverless cluster, to allow web-based applications
            to access the cluster over a secure HTTP endpoint.
            ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html))

            1. All calls to the Data API are **synchronous**. By default, a
                call times out if it's not finished processing within **45
                seconds**. However, you can continue running a SQL statement
                if the call times out by using the **continueAfterTimeout**
                parameter.

            2. Users don't need to pass credentials with calls to the Data
                API, because the Data API uses database credentials stored in
                Secrets Manager.

            3. To store credentials in Secrets Manager, users must be granted
                    the appropriate permissions to use Secrets Manager, and also
                    the Data API.

            4. You can also use Data API to integrate Aurora Serverless with
                Lambda, AppSync, and Cloud9.

            5.  The API provides a more secure way to use Lambda. It enables you
                to access your DB cluster without you needing to configure a
                Lambda function to access resources in a VPC.

82. Logging

    1. By default, error logs for Aurora Serverless are enabled and
        automatically uploaded to CloudWatch.

    2. You can also have your Aurora Serverless DB cluster upload
        Aurora database-engine specific logs to CloudWatch by using a
        **custom DB cluster parameter group**.

83. Potential issue of Aurora Serverless cold start to APIG + Lambda
    ([Ref](https://dev.to/dvddpl/how-to-deal-with-aurora-serverless-coldstarts-ml0))

    1. Since a sleeping Aurora DB cluster can take up to **25 secs** to
        be awakened, could hit the hard limit on API Gateway endpoint
        (max 29 secs)
        ([Ref](https://docs.aws.amazon.com/apigateway/latest/developerguide/limits.html))

84. Aurora Serverless **Query Editor** (in console) allows to run any
    valid SQL statement on the Aurora Serverless DB cluster, including
    data manipulation and data definition statements.

85. For Aurora Serverless, any enabled logging is automatically
    published to CloudWatch.

86. Limitations

    1. Single AZ

    2. Cannot use **custom endpoints** for **Aurora Serverless**
        clusters.

    3. Cannot be used with Parallel Query.

87. A company would like to use a third-party vendor SaaS product to
    perform data analytics on data stored inside an **Aurora
    Serverless cluster**. What is the simplest, and most secure
    solution to integrate the SaaS product with the Aurora cluster?

    1. Enable Data API on the Aurora Serverless cluster, to allow
        web-based applications to access the cluster over a secure
        HTTP endpoint.

    2. NO: Create a VPC endpoint service inside the Aurora Serverless
        cluster's VPC using AWS PrivateLink

        1. Not the simplest option. It would require provisioning and
            network configuration of PrivateLink.

    3. NO: Create a VPC SG rule allowing inbound traffic from the SaaS
        product IP range. Apply the SG to the Aurora Serverless
        cluster's VPC endpoint.

        1. Not the simplest and most secure option as it opens public
            internet traffic to the private VPC.

    4. NO: Create a Site-to-site VPN connection from Amazon Aurora
        Serverless cluster's VPC to the SaaS product vendor's
        network.

        1. Not the simplest, cost-efficient, and secure option.
            Configuring VPN connection may not be possible. Even if
            possible, it is a complex implementation. It is also not
            recommended from a security point of view because it
            connects the two networks.

---
## DynamoDB

88. Fully managed key-value database on SSDs (solid-state disks),
    delivers single-digit millisecond performance at any scale.

89. Max item size: **400 KB**; **Unlimited** data volume; Interface:
    **AWS API**

90. Multi-AZ: Replicated in 3 AZs by default.

91. Multi-Region: DynamoDB Global Tables = [Multi-Region
    Replication](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html)
    = [Cross-Region
    Replication](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.CrossRegionRepl.html)
    ([Ref-1](https://aws.amazon.com/dynamodb/global-tables/),
    [Ref-2](https://aws.amazon.com/blogs/database/how-to-use-amazon-dynamodb-global-tables-to-power-multiregion-architectures/))

    1. **Global Tables** automatically replicate data across 2 or more
        Regions, with full support for multi-master writes.

    2. To create a global table

        1. Enable DynamoDB Streams**.**

        2. Ensure that this table is empty**.**

        3. Ensure that a table with the same name does not exist in
             the selected Region.

    3. Should also configure **DynamoDB auto scaling** for the global
        tables.

    4. Capacity planning should account for additional write capacity
        associated with automatic multi-master replication.

    5. Transactions are ACID-compliant only in the Region where the
        write is made originally.

92. Strongly consistent reads/writes vs. eventually consistent
    reads/writes

    1. DynamoDB uses eventually consistent reads, unless you specify
        otherwise**.**

    2. For Read operations (**GetItem**, **Query**, **Scan**) set
        **ConsistentRead** to **true** for strongly consistent reads.

    3. Strongly consistent reads use more throughput capacity than
        eventually consistent reads.

    4. Strongly consistent reads may have higher latency than
        eventually consistent reads.

    5. Strongly consistent reads are not supported on GSI.

93. Read/Write Capacity Unit of Provisioned (= Read/Write Request Unit
    of On-demand) = (RCU, WCU below)

    1. DynamoDB read and write speeds are determined by the number of
        available partitions.

    2. A **partition** can support a maximum of **3000 RCUs, 1000 WCUs,
        10GB of data**.

        1. For 5500 RCUs and 1500 WCUs: (5500/3000) + (1500/1000) =
            3.333 -4 partitions

        2. For 45 GB of data: (45/10) = 4.5 -5 partitions

    3. **Default Throughput (per table)
        [Quotas](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Limits.html#default-limits-throughput)**:
        **40000 RCU, 40000 WCU**

    4. **1 RCU** provides **1 strongly consistent read**, or **2
        eventually consistent reads** per second for item **<= 4KB**.

    5. **2 RCUs** provide **1 transactional read** per second for item
        **<= 4 KB**.

    6. **1 WCU** provides **1 write** per second for item **<= 1KB**.

    7. **2 WCUs** provide **1 transactional write** per second for item
        <= **1 KB**.

    8. Adding/updating/deleting an item in a table also costs a WCU and
        additional WCUs to write to any LSI and GSI.

        1. E.g. 6 RCUs and 6 WCUs
            ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html))

            1.  6 strongly consistent read x 4 KB = 24 KB per second

            2.  12 eventually consistent read x 4 KB = 48 KB per second

            3.  6 write x 1 KB = 6 KB per second

            4.  3 transactional read x 4 KB = 12 KB per second

            5.  3 transactional write x 1 KB = 3 KB per second

        2. E.g. Item size 8 KB

            1.  2 RCUs for 1 strongly consistent read per second,

            2.  1 RCU for 1 eventually consistent read per second, or

            3.  4 RCUs for 1 transactional read per second.

        3. E.g. Item size 10KB; for reading 80 strongly consistent
             items from a table per second.

             1.  How many RCUs in 10KB: 10KB / 4KB = 2.5 =3

             2.  Each item requires 3 RCUs; 80 items need: 3 * 80 = 240
                 RCUs. Strongly consistent reads

             3.  Eventually consistent reads =120 RCUs

94. On-Demand Capacity Mode
    ([Ref-1](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html))

    1. Best solution for applications whose database workload is
        complex to forecast, with large spikes of short duration, or
        average table utilization well below the peak.

    2. On-demand mode instantly accommodates up to double the previous
        peak traffic on a table.

    3. However, **throttling** can occur if you exceed double your
        previous peak within **30 minutes**.

    4. Newly created table with on-demand capacity mode:

        1. **(Default) Previous peak = 6000 RCUs or 2000 WCUs**

        2. You can drive up to double the **previous peak**
            immediately, which enables newly created on-demand tables
            to serve up to **12000 RCUs or 4000 WCUs**, or any linear
            combination of the two.

    5. Existing table switched to on-demand capacity mode:

        1. **Previous peak** = max (half the previous max **WCUs** and
            max **RCUs** provisioned, **default previous peak**).

95. Provisioned Capacity Mode
    ([Ref-1](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html))

    1. Best solution for predictable, consistent application traffic.

    2. **Burst Capacity**: DynamoDB retains up to **5 mins** of unused
        RCU/WCU capacity; but not guaranteed.
        ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html#bp-partition-key-throughput-bursting))

    3. **DynamoDB Auto Scaling**
        ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html))

        1. **DynamoDB Auto Scaling** uses **Application Auto Scaling**
            to dynamically adjust **provisioned throughput capacity**
            in response to actual traffic patterns.

        2. This enables a **table** or a **GSI** to increase/decrease
            its provisioned read and write capacity to handle sudden
            change of traffic, without throttling.
            ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/AutoScaling.html))

        3. If you use the console to create a table or a GSI,
             **DynamoDB auto scaling** is enabled by default.

    4. Scale without downtime but takes time. Decreases in throughput
        will take anywhere from a few seconds to a few minutes, while
        increases in throughput will take anywhere from a few minutes
        to a few hours.

    5. The smallest amount of Reserved Capacity that one can buy: **100
        RCU** and **100 WCU**

96. HTTP **400** (Bad Request) +
    **ProvisionedThroughputExceededException**: (Throttling exception)

    1. Exceeding provisioned throughput limit (RCU/WCU per partition)

    2. Might be trying to delete, create, or update tables too quickly.

    3. Using one partition or key extensively.

    4. Stale or unused data occupying partition and key space.

    5. Your request rate is too high. The AWS SDKs for DynamoDB
        automatically retry requests that receive this exception. Your
        request is eventually successful, unless your retry queue is
        too large to finish. Reduce the frequency of requests and use
        exponential backoff.

97. HTTP **403**

    1. The request must contain either a valid (registered) AWS access
        key ID or X.509 certificate.

    2. The X.509 certificate or AWS access key ID provided does not
        exist in our records.

    3. The AWS access key ID needs a subscription for the service.

98. HTTP **500** (InternalFailure)

    1. When network delays or network outages occur, **strongly
        consistent reads** on a DynamoDB table are more likely to fail
        and return a 500 error.

    2. The request processing has failed because of an unknown error,
        exception or failure.

99. DynamoDB Streams

    1. Manage Stream | View Type (cannot be changed after it's
        created)

        1. **Keys only** - only the key attributes of the modified item

        2. **New image** - the entire item, as it appears after it was
            modified

        3. **Old image** - the entire item, as it appeared before it
             was modified

        4. **New and old images** - both the new and the old images of
            the item

100. Automatic Backup: optional; aka PITR (point-in-time recovery)
     ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/PointInTimeRecovery_Howitworks.html))

     1. Retention period: **fixed 35 days** when enabling **PITR
         (point-in-time recovery)**

         1. To retain data longer than 35 days, enable **DynamoDB
             Streams** on the DynamoDB table. Create a **Kinesis
             Firehose Stream** to load the data into an S3 bucket.
             Create a Lambda function to poll the DynamoDB stream and
             deliver batch records from streams to Firehose.

     2. The PITR process always restores to a new table, without
         consuming any provisioned throughput on the table.

     3. Any number of users can execute up to **4** concurrent restores
         (any type of restore) in a given account.

     4. You can do a **full table restore** using PITR, or you can
         configure the **destination table settings** on the restored
         table: GSIs, LSIs, Billing mode, Provisioned read and write
         capacity, Encryption settings.

     5. A **full table restore** restores table data to that point in
         time, but all table settings for the restored table come from
         the current settings of the source table at the time of the
         restore.

     6. You can restore your DynamoDB table data **across Regions**
         such that the restored table is created in a different Region
         from where the source table resides.

     7. Restores can be faster and more cost-efficient if you exclude
         some or all indexes from being created on the restored table.

     8. You must manually set the following on the restored table:

         1. Auto scaling policies

         2. Time to Live (TTL) settings

         3. Point-in-time recovery settings

         4. Stream settings

         5.  Tags

         6.  IAM policies

         7.  CloudWatch metrics and alarms

     1. **DynamoDB table export** allows exporting data from any time
         within your PITR window to a S3 bucket.

101. On-demand Backup and Restore
     ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/BackupRestore.html))

     1. You can restore the table to the same Region or to a different
         Region from where the backup resides.

     2. You can exclude secondary indexes from being created on the new
         restored table.

     3. You cannot create new indexes at the time of restore.

     4. You can specify a different encryption mode.

     5. While a restore is in progress, don't modify or delete your
         IAM role policy, to avoid any unexpected behavior.

     6. If your backup is encrypted with an AWS managed CMK or a
         customer managed CMK, don't disable or delete the key while
         a restore is in progress, or the restore will fail. After the
         restore operation is complete, you can change the encryption
         key for the restored table and disable or delete the old key.

102. DynamoDB does **not** have the **deletion protection** feature.

103. DynamoDB **Time to Live (TTL)** lets you define when items in a
     table expire so that they can be automatically deleted from the
     database. You can set the expiration date in **epoch** format in
     the **TTL attribute** on a **per-item basis**.

     1. To limit storage usage to only those records that are relevant,
         or must be retained only for a certain amount of time (e.g.
         to enforce compliance and auditing requirements).

     2. To ensure unauthorized updates to the **TTL** attribute are
         prevented,

         1. When configuring DynamoDB table TTL, specify authorized
             users ARNs.

         2. Use IAM policies to deny update actions to the TTL
             attribute or feature configuration. Create an IAM role
             policy that allows **dynamodb:UpdateTimeToLive**. Assign
             the role policy to the authorized users.

     3. Enable **DynamoDB Streams** on the table for processing items
         deleted by TTL.

104. **DAX** (DynamoDB Accelerator)

     1. DAX is a fully managed in-memory cache for read-heavy workloads
         with response times in microseconds for millions of requests
         per second; no need to manage cache invalidation, data
         population, or cluster management.

     2. DAX only allows queries on the same partition key as the base
         table.

     3. DAX is a DynamoDB specific caching solution and does NOT offer
         cross-region replication.

     4. If the DAX node sizes are too small, you will receive
         **ThrottlingException** if the number of requests sent to DAX
         exceeds the capacity of a node.

105. **DynamoDB transactions**
     ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/transactions.html))

     1. With the transaction write API, you can group multiple Put,
         Update, Delete, and ConditionCheck actions. You can then
         submit the actions as a single **TransactWriteItems**
         operation that either succeeds or fails as a unit.

     2. You can group multiple Get actions and submit as a single
         **TransactGetItems** operation.

     3. Enable automatic scaling on your tables, or ensure that you
         have provisioned enough throughput capacity to perform the 2
         read or write operations for every item in your transaction.

     4. Avoid using transactions for ingesting data in bulk. For bulk
         writes, it is better to use **BatchWriteItem**.

106. Each query can only use one index. If you want to query and match
     on two different columns, you need to create an index that can do
     that properly.

107. When you write your queries, you need to specify exactly which
     index should be used for each query.

     1. GSI lets you query across the entire table to find any record
         that matches a particular value.

     2. LSI can only help finding data within a single partition key.

108. Primary Keys and Secondary Indexes

     1. Simple primary: **Hash key** **= partition key**, allowed data
         types: **string, number, binary**

     2. Composite primary: **Hash and range key = partition key and
         sort key**

     3. These keys are used for splitting the data across partitions.

     4. Each partition is replicated across the 3 AZs automatically.

     5. **LSI (Local secondary index)**

         1. The primary key of a LSI must be **composite** (partition
             key and sort key).

         2. Same partition key as the base table but a different sort
             key; same attributes

         3. Support **eventual consistency** and **strong
              consistency**.

         4. With a LSI, there is a limit on **item collection sizes**:
             For every distinct partition key value, the total sizes
             of all table and index items cannot exceed **10 GB**.

         5.  Up to **5** LSI per table.

         6.  Can be created at the time of table creation; cannot be
             modified or deleted.

         7.  LSIs count against the provisioned throughput
              (performance) of the DynamoDB table.

     6. **GSI (Global secondary index)**

         1. The primary key of a GSI can be either simple (partition
             key) or composite (partition key and sort key).

         2. Can use a different partition key (and different sort key
             if present) from those from the base table.

         3. Like adding another DynamoDB table which gets changes
              propagated from the base table.

         4. GSI supports **eventually consistent reads**; cannot do
             **strongly consistent reads**.

             1.  If you need strongly consistent reads, you need to
                 create a new DynamoDB table instead.

         5.  Default max **20** GSI per table. No size restrictions.

         6.  Can be added to existing tables.

         7.  Consistency lags behind the base table.

         v3. Limit the projected attributes.

         ix. GSIs have their own provisioned throughput independent of
             the main table.

         x.  If GSIs run out of provisioned throughput, the main table
             will be throttled, not just the GSIs.

     7. If you want to create more than one table with secondary
         indexes, you must do so **sequentially**. For example, you
         would create the first table and wait for it to become
         ACTIVE, create the next table and wait for it to become
         ACTIVE, and so on.

         1. If you try to concurrently create more than one table with
             a secondary index, DynamoDB returns a
             **LimitExceededException**.
             ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SecondaryIndexes.html))

109. Encryption-at-rest is always on and cannot be disabled.

     1. Tables can be encrypted under

         1. an **AWS owned CMK** **(default;** AWS/DynamoDB **service
             default CMK)**,

         2. an **AWS managed CMK** for DynamoDB in your account.

         3. **NOT** support **Customer managed CMKs**.

     2. The CMK is used to create a data key (**table key**) unique to
         each table. The **table key** is managed by DynamoDB and
         **stored with the table** in an encrypted form.

     3. DynamoDB encrypts **every item** with a **data encrypted key**.
         The **data encrypted key** is encrypted with the **table
         key** and stored with the data.

     4. **Table keys** are cached for up to **12 hours** in plaintext
         by DynamoDB, but a request is sent to KMS after **5 minutes**
         of table key inactivity to check for permission changes.

     5. **CloudTrail logs** can be used to audit KMS CMKs (Customer
         Master Keys) and identify DynamoDB tables that are using the
         keys for encryption at rest.

         1. When DynamoDB uses a CMK for server-side encryption, it
             uses KMS API operations (e.g. GenerateDataKey, Decrypt,
             CreateGrant) which are logged in CloudTrail logs.

110. Encryption-in-transit using **dynamodb encryption client
     libraries**

     1. You should never encrypt the primary key attributes, this is to
         prevent your DynamoDB from running a table scan to find a
         single item. Instead, you should use **Sign only** as an
         attribute action for your partition and sort keys.

111. IAM can restrict which tables/streams a user can access, and which
     ITEMS and Attributes a user can access.

112. Logging and Monitoring

     1. When **CloudWatch Contributor Insights** is enabled on a table
         or GSI, DynamoDB creates the following rules on your behalf:
         Most accessed items, Most throttled keys.
         ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/contributorinsights_HowItWorks.html))

     2. CloudWatch metrics

         1. ConsumedReadCapacityUnits - RCUs per time period used

         2. ConsumedWriteCapacityUnits - WCUs per time period used

         3. ReadThrottleEvents - Provisioned RCU threshold breached

         4. WriteThrottleEvents - Provisioned WCU threshold breached

         5.  ThrottledRequests - ReadThrottleEvents +
             WriteThrottleEvents

113. DynamoDB updates the **Storage Size** value approximately only
     every **6 hours**.

114. DynamoDB updates the **ItemCount** value approximately every **6
     hours**.

115. PutItem, UpdateItem, and DeleteItem have a **ReturnValues**
     parameter, which you can use to return the attributes before or
     after they are modified.

116. Attributes can be scalars, JSON, XML

117. Projection Type

     1. KEYS_ONLY - Only the index and primary keys are projected
         (smallest index - more performant)

     2. INCLUDE - Only the specified attributes are projected.

     3. ALL - All attributes are projected (biggest index - least
         performant).

118. **Data plane** operations such as reads and writes to a table often
     have higher availability design goals than **control plane**
     operations such as changing the table metadata.

119. Best Practices
     ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html))

     1. **get-item** retrieves an item by its primary key. GetItem is
         highly efficient as it provides direct access to the physical
         location of the item.

     2. Use **get-item** with **Project expression** to reduce the size
         of the read operations and increase read efficiency.

     3. If a **scan** operation on a large DynamoDB table is taking a
         long time to execute, **Parallel Scans** can be used to
         decrease the execution time of the scan operation.

     4. **Conditional expressions** allow a condition to be checked
         **before** an **update-item** is applied.

     5. Store metadata in DynamoDB and blobs in S3.

     6. For storing time series data, use a table per day, week, month
         etc.

         1. If we put all the time series data in one big table, the
             last partition is the one that gets all the read and
             write activity, limiting the throughput.

         2. If we create a new table for each period, we can maximize
             the RCU and WCU efficiency against a smaller number of
             partitions.

     7. Recommended approach to store ***date*** for querying data in a
         DynamoDB table by date range:
         ([Ref-1](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.DataTypes.html),
         [Ref-2](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.NamingRulesDataTypes.html#HowItWorks.DataTypes))

         1. Use the **String** data type and ISO-8601 formatted
             strings.

     8. To locate all items in a DynamoDB table with a particular sort
         key

         1. Scan against a table, with filters. But it is inefficient.

         2. Query with a GSI. GSIs will allow a new index with the sort
             key as a partition key, and query will work.

             1.  LSIs can't be used, they only allow an alternative sort
                 key and query can only work against 1 partition key,
                 with a single or range of sort.

             2.  GetItem won't work, it needs a single Partition-Key
                 and Sort-Key.

     1. To truncate a DynamoDB table,

         1. Use **aws dynamodb scan** to scan the table, iterate
             through all keys and use **aws dynamodb delete-item** to
             delete each item.

         2. To delete multiple items, use the **batch-write-item**
             instead of **delete-item**.

         3. Note that both **scan** operation and **batch-write-item**
              operations consume **RCUs and WCUs**.

         4. If there are no requirements to keep the table, a faster
             and more cost-effective approach would be to delete and
             recreate the entire DynamoDB table.

     10. To maintain a record of all **GetItem** and **PutItem**
         operations performed on a DynamoDB table for audit purposes.

         1. DynamoDB **data-plane** API operations (e.g. GetItem,
             PutItem) are **not** logged into CloudTrail logs.

         2. Enable **DynamoDB Streams**; DynamoDB streams record events
             every time any table item is modified. Use Lambda
             function to read, process and record these stream
             records.

     11. To increase the speed of read operations:

         1. **Secondary Indexes**, which contain a subset of attributes
             from a table, and an alternate key for queries

         2. **DAX**, which works as an in-memory cache in front of
             DynamoDB.

     12. What are some effective ways to handle extreme write burstyness
         on a table (e.g. black friday/xmas sales)

         1. Set an appropriate WCU on the table for nominal load and
             then utilise SQS as a queue, to smooth any peaks, writing
             the data to DDB when conditions allow.

     13. What are some effective ways to handle extreme read burstyness
         on a table (e.g. black friday/xmas sales)

         1. Set an appropriate RCU on the table for nominal load and
             then utilise ElastiCache, S3, lambda to generate static
             content to offload DB read loads.

         2. Utilise the burst capacity of a table, sparingly, which
             provides enough capacity to remove any peaks in demand.

     14. What is the optimal method for importing a CSV file from S3
         into the DynamoDB table?
         ([Ref](https://aws.amazon.com/blogs/database/implementing-bulk-csv-ingestion-to-amazon-dynamodb/))

         1. Use Lambda Function to read the file and import the data
             into the DynamoDB table (the simplest and most
             cost-efficient method).

         2. AWS CLI can only import JSON formatted data into DynamoDB
             tables.

         3. AWS Management Console only allows entering single items
              into DynamoDB tables.

         4. Data Pipeline is not the optimal solution. It requires
             non-trivial activities and costs associated with
             configuration of Data Pipelines and EMR cluster
             infrastructure.

     15. If after recreating a DynamoDB table with new LSI, the
         application starts to experience throttling when performing
         write operations on the base table even though the number of
         table updates has not increased, it could be because:

         1. LSI shares write-capacity with the base table. Performing
             write operations to a table causes the LSI to be also
             updated. This consumes write capacity units from the base
             table. In this case, the addition of the LSI is causing
             the provisioned write capacity to be exceeded.

     16. A DynamoDB table is currently configured for provisioned
         capacity mode. The company has just introduced a popular line
         of products. This is generating large volumes of data, with
         traffic peaking at over 4,000 reads per second. Reports are
         coming in that the application is slow and is occasionally
         returning outdated data. (3)

         1. A partition can support a maximum of 3000 RCUs, 1000 WCUs,
             10GB of data.

         2. A partition is automatically split, creating two new
             partitions. The current throughput of the original
             partition is divided among the new partitions. This can
             lead to throttling if the throughput is insufficient.

         3. The partition key is no longer effective and is resulting
              in hot partitions.

     17. Using **Web Identity Federation (WIF)**
         ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WIF.html))

         1. If you are writing an application targeted at large numbers
             of users, you can optionally use **Web Identity
             Federation** for authentication and authorization.

         2. The app calls a third-party **identity provider** (e.g.
             Amazon, Facebook, Google) to authenticate the user and
             the app. The identity provider returns a **web identity
             token** to the app.

         3. The app calls **AWS STS** (Security Token Service) and
              passes the **web identity token** as input. **AWS STS**
              authorizes the app and gives it **temporary AWS access
              credentials**.

         4. The app is allowed to assume an IAM role (e.g. GameRole)
             and access AWS resources in accordance with the role's
             security policy.

             1.  The **Condition** clause determines which items in the
                 GameScores table are visible to the app. It does this
                 by comparing the Login (e.g. Amazon ID) to the UserId
                 partition key values in GameScores.

             2.  Only the items **belonging to the current user** can be
                 processed using one of DynamoDB actions that are
                 listed in this policy. Other items in the table
                 cannot be accessed.

             3.  Furthermore, only the **specific attributes** listed in
                 the policy can be accessed.
                 ([Ref](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WIF.html))

                    "Statement":
                    {
                        "Sid": "AllowAccessToOnlyItemsMatchingUserID",
                    >
                    "Effect": "Allow",
                    >
                    "Action": (
                    >
                    "dynamodb:GetItem",
                    >
                    "dynamodb:Query",
                    >
                    "dynamodb:PutItem",
                    >
                    "dynamodb:UpdateItem"
                    >
                    ),
                    >
                    "Resource": (
                    "arn:aws:dynamodb:us-west-2:123456789012:table/GameScores" ),
                    >
                    "Condition": {
                    >
                    "ForAllValues:StringEquals": {
                    >
                    "dynamodb:LeadingKeys": (
                    >
                    "${www.amazon.com:user_id}"
                    >
                    ),
                    >
                    "dynamodb:Attributes": ( "UserId", "GameTitle", "Wins",
                    "Losses" )
                    >
                    },
                    >
                    "StringEqualsIfExists": {
                    >
                    "dynamodb:Select": "SPECIFIC_ATTRIBUTES"
                    >
                    }
                    >
                    }
                    >
                    }
                    >
                    )

---
## EC2 Instance Databases

120. For a relational database deployed to EBS-backed EC2 instance

     1. **RAID 0** is used to distribute I/O across volumes and achieve
         increased IOPS and throughput performance.
         ([Ref](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/raid-config.html))

         1. Choose RAID 0 when I/O performance is more important than
             fault tolerance; e.g. as in a heavily used database
             (where data replication is already set up separately).

     2. **RAID 1** is used to simultaneously write data to multiple
         volumes thus providing increased data redundancy.

     3. AWS does not recommend RAID 5 and RAID 6 for EBS volumes.

121. A database being migrated to EC2 instances requires 10000 IOPS.
     Which [EBS Volume
     type](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
     should be used?

     1. gp2 / gp3 volume type provides max 16,000 IOPS

     2. io2 (max 256,000 IOPS), io1 (max 64,000 IOPS), st1 (max 500
         IOPS), sc1 (max 250 IOPS)

122. Instances

     1. Memory optimized (R5) - memory-intensive apps, high performance
         DB, distributed in-memory caches

     2. Storage optimized (I3) - high-volume IOPS requiring
         low-latency, internet-scale non-relational databases

     3. Burstable performance (T3) - consistent cost for unpredictable
         workloads, smaller DB with spiky usage

123. Storage

     1. Provisioned IOPS SSD - I/O apps needing consistent low latency,
         large databases

     2. General purpose SSD - default type, small to medium sized DB,
         dev/test env, boot volumes

     3. Magnetic volumes - least expensive EBS volume type, infrequent
         data access

---
## Migrating Databases

124. **DMS** is a tool that performs data migration during a database
     migration process.

     1. ME: When using DMS to replicate RDS from account 1 region A to
         Redshift in account 2 region B, should the replication
         instance in account 1 or 2, region A or B?

     2. **DMS** can be used for continuous data replication with high
         availability.

     3. The source database remains fully operational during the
         migration.

     4. Sources -DMS (Task on replication instance (EC2)) -> Targets
         ([Ref](https://docs.aws.amazon.com/dms/latest/userguide/Welcome.html))

     5. Either the source or the target database (or both) need to
         reside in RDS or on EC2 (i.e. not on-prem to on-prem).

     6. If the replication instance is in a storage-full state, the
         migration tasks may fail without an error message.

         1. Increase the allocated storage for the replication
             instance.

     7. [Sources](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.html)

         1. **On-prem/EC2 instance databases**

             1.  Oracle, SQL Server, MySQL, MariaDB, PostgreSQL,
                 MongoDB, SAP ASE, **Db2 LUW**

         2. **Azure SQL Database** (support full data load, not support
             Change data capture (CDC))

         3. **RDS, DocumentDB, S3** (support full data load and change
              data capture (CDC) when using S3 as a source)

     8. [Targets](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.html)

         1. **On-prem/EC2 instance databases**

             1.  Oracle, SQL Server, PostgreSQL, MySQL, MariaDB,
                 MongoDB, SAP ASE

         2. **RDS, S3, DynamoDB, Redshift, Elasticsearch Service,
             Kinesis Data Streams, MSK, self-managed Kafka,
             DocumentDB, Neptune**

     1. DMS **does not** support **cross-region migration** for the
         following target endpoint types:

         1. DynamoDB, Elasticsearch Service, Kinesis Data Streams

125. **SCT** is a tool for accessing schema conversion and converting
     existing database schema from one database engine to another.

     1. **SCT** uses AWS access key and secret key for AWS
         authentication.

126. **DMS** vs. **SCT**

     1. **DMS** traditionally moves smaller relational workloads (<10
         TB) and MongoDB, whereas **SCT** is primarily used to migrate
         large data warehouse workloads.

     2. **DMS** supports ongoing replication to keep the target in sync
         with the source; **SCT** does not.

127. **WQF (Workload Qualification Framework)** with SCT

     1. WQF is a standalone tool that is used during the database
         migration planning phase to assess migration workloads. It
         produces an assessment report detailing migration complexity
         and size, and provides migration strategy and tool
         recommendations.
         ([Ref](https://docs.aws.amazon.com/SchemaConversionTool/latest/userguide/CHAP-WQF.html))

     2. Classify OLTP and OLAP workloads using SCT with WQF

     3. You can use WQF for the following migration scenarios:
         ([Ref](https://aws.amazon.com/blogs/database/classify-oltp-and-olap-workloads-using-aws-sct-with-workload-qualification-framework-wqf/))

         1. Oracle to RDS PostgreSQL or Aurora PostgreSQL

         2. Oracle to RDS MySQL or Aurora MySQL

         3. SQL Server to RDS PostgreSQL or Aurora PostgreSQL

         4. SQL Server to RDS MySQL or Aurora MySQL

128. **DataSync** is a data migration service for transferring data from
     on-premise file storage sources to AWS services such as S3 and
     EFS

129. Conversion of stored procedures

     1. **SCT** automates the conversion of Oracle PL/SQL and SQL
         Server T-SQL code to equivalent code in the Aurora / MySQL
         dialect of SQL or the equivalent PL/pgSQL code in PostgreSQL.

     2. When a code fragment cannot be automatically converted to the
         target language, SCT will clearly document all locations that
         require manual input from the application developer.

130. DMS can be used with Snowball Edge and S3 to migrate large
     databases.
     ([Ref](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_LargeDBs.html))

     1. You use the **SCT** to extract the data locally and move it to
         an Edge device.

     2. You ship the Edge device or devices back to AWS.

     3. After AWS receives your shipment, the Edge device automatically
         loads its data into a S3 bucket.

     4. DMS takes the files and migrates the data to the target data
         store. If you are using **change data capture (CDC)**, those
         updates are written to the S3 bucket and then applied to the
         target data store.

131. A DMS replication instance had unintentionally been configured to
     be publicly accessible. Access to the DMS replication instances
     should only be available to a host within the same VPC. To fix
     the issue, either:

     1. Change the subnets associated with the replication instance
         from public to private, OR

     2. Delete the replication instance and all tasks using it before
         recreating it.

Migration

132. Physical migration means that physical copies of database files are
     used to migrate the database.

133. Logical migration means that the migration is accomplished by
     applying logical database changes, such as inserts, updates, and
     deletes.

134. Migrate from Aurora to Aurora Serverless and vice versa
     ([Ref](https://aws.amazon.com/rds/aurora/faqs/))

     1. Restore a snapshot taken from an existing Aurora provisioned
         cluster into an Aurora Serverless DB Cluster (and vice
         versa).

135. Migrate from RDS to Aurora MySQL
     ([Source](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Migrating.html))

     1. Migrate from a RDS MySQL DB **Snapshot** to an Aurora MySQL DB
         **cluster**.
         ([Source](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Migrating.RDSMySQL.html))

     2. Migrate from a RDS MySQL DB **instance** by **creating an
         Aurora MySQL Read Replica**.
         ([Source](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Migrating.RDSMySQL.Replica.html))

         1. When the **replica lag** between the MySQL DB instance and
             the Aurora MySQL Read Replica is **0**, you can direct
             your client applications to read from the Aurora Read
             Replica and then stop replication to make the Aurora
             MySQL Read Replica a **standalone** Aurora MySQL DB
             cluster for reading and writing.

136. Migrate from RDS to Aurora PostgreSQL
     ([Source](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Migrating.html))

     1. Migrate from a RDS PostgreSQL DB **Snapshot** to an Aurora
         PostgreSQL DB cluster
         ([Source](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Migrating.html#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Import.Console))

     2. Migrate from a RDS PostgreSQL DB instance by **creating an
         Aurora PostgreSQL Read Replica**.
         ([Source](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Migrating.html#AuroraPostgreSQL.Migrating.RDSPostgreSQL.Import.Console))

         1. When the **replica lag** between the PostgreSQL DB instance
             and the Aurora PostgreSQL Replica is **0**, you can
             promote the Aurora Replica to be a standalone Aurora
             PostgreSQL DB cluster.

     3. Import data from S3 into a table belonging to an Aurora
         PostgreSQL DB cluster using the
         **aws_s3.table_import_from_s3** function.
         ([Source](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Migrating.html#USER_PostgreSQL.S3Import))

137. Migrate from external MySQL database to Aurora MySQL

     1. Only if your database supports the **InnoDB** or **MyISAM**
         tablespaces.

     2. If the external MySQL database uses **memcached**, remove
         **memcached** before migrating it.

     3. [Option
         1](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Migrating.ExtMySQL.html#AuroraMySQL.Migrating.ExtMySQL.mysqldump):
         Use **mysqldump** utility to create a dump of your data, then
         import data into an existing Aurora MySQL DB cluster.
         **mysqldump** utility uses DDL and DML statements to recreate
         the database schemas and load the data, it is a very time
         consuming process.

     4. [Option
         2](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Migrating.ExtMySQL.html#AuroraMySQL.Migrating.ExtMySQL.S3):
         Use **xtracbackup** (**Percona XtraBackup** utility) to
         create backup files from the source database to a S3 bucket,
         and then restore an Aurora MySQL DB cluster from those files
         using the "**Restore from S3**" option (considerably faster
         than using **mysqldump**).

     5. [Option
         3](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.LoadFromS3.html):
         Save data from your database as text files and copy those
         files to an S3 bucket. Then load that data into an existing
         Aurora MySQL DB cluster using the **LOAD DATA FROM S3** MySQL
         command.

         1. Use **LOAD XML FROM S3** SQL statement to import the XML
             file from S3 to the database.

138. To plan a database conversion and migration from on-premise
     database cluster to AWS, a **database migration assessment
     report** shall be created to assess the migration compatibility,
     task, effort, and recommendations.

     1. Install the **SCT Tool** on a virtual machine with access to
         on-prem network.

     2. Install **database drivers** required to connect to the source
         and target database.

139. Migrate from a EC2-based MySQL database to RDS MySQL database

     1. How to perform data validations on the target database with the
         most operational efficiency?

         1. Set **ValidationOnly** to true for a new task in DMS.

     2. The database is fairly large in size and has an uptime
         requirement near 24X7. There are a significant number of
         tables in the MySQL database that cannot be migrated due to
         company regulations. Requirements: rapid implementation and
         minimal downtime. (3)

         1. Create a new RDS MySQL database. Use the **DMS table
             mapping function** to ensure that only the specified
             tables are migrated. Ensure that the migration is
             performed in **change data capture (CDC)** mode.

         2. Should not use Read replica - Read replica copies all
             tables, so confidential information would be copied.

         3. Should not use Snapshot - Snapshots extract all tables and
              increase downtime.

         4. mysqldump can dump some tables when used with
             --ignore-table, but the downtime would be longer.

140. Migrate from Cassandra cluster to DynamoDB

     1. Install **SCT Data Extraction Agent**. SCT data extraction
         agents are required when the source and target databases are
         significantly different and require complex data
         transformations.

     2. Create a **Clone Datacenter**. The extraction process performed
         by the data extraction agents imposes additional overhead
         load on the Cassandra database. In order to avoid having a
         negative impact on the operation and performance of the
         source production cluster, a clone datacenter containing a
         copy of the production data should be created. SCT should be
         configured to connect to this clone cluster.

         1. The data extraction agents should NOT be connected to the
             production Cassandra cluster.

     3. Create an **S3 bucket**. S3 bucket is required to store the
         output of the data extraction agent conversion and migration
         process.

141. Migrate from MongoDB cluster to DocumentDB

     1. To perform an online data migration from on-premise MongoDB
         cluster to DocumentDB in order to minimize source database
         downtime, DMS is being used to perform data migration. Before
         beginning the full load of migration data
         ([Ref](https://aws.amazon.com/blogs/database/migrating-to-amazon-documentdb-with-the-online-method/))

         1. Create indexes on the target database. Because DMS does not
             migrate indexes. For large data migrations, it is most
             efficient to pre-create indexes in the target DocumentDB
             cluster before migrating the data.

         2. Recommend to use DMS on the secondary node of source
             cluster (replica set) for the read operations as it can
             result in reducing the overall load on the primary
             instance.

         3. Turn on **CDC** (**change data capture**) on the source
              database. CDC is required in online data migration. DMS
              uses CDC to replicate changes to target DocumentDB.

         4. DMS uses the source **MongoDB oplog** to perform data
             migration. During an online data migration, the **oplog**
             on each replica set should be ***large enough*** to
             contain all changes made during the entire duration of
             the data migration process.

     2. To move a 15TB MongoDB database to a DocumentDB and should be
         fast and incur minimum downtime: (3)

         1. Migrate the database with a hybrid approach, using the
             **mongodump** and **mongorestore** tools for an initial
             full load and DMS with **change data capture (CDC)** to
             replicate changes.

                (The hybrid approach balances migration speed and downtime).

     3. Migrate the database with an **online** approach, using DMS with a
        **full load** and **CDC strateg**y (minimum downtime but slow)

     4. Migrate the database with an **offline** approach, using DMS with a
        **full load** strategy (fast migration but long downtime)

142. Migrate from SQL based database to DynamoDB

     1. To refactor and migrate an application from using a relational
         SQL based database to a NoSQL DynamoDB database, what is the
         first action should do when designing the application data
         model for DynamoDB?

         1. Identify database query access patterns.

143. Migrate from a RDS Oracle to Aurora MySQL

     1. Create the target database.

     2. Run the conversion report using SCT and execute generated
         scripts.

     3. Resolve any issues from the SCT stage.

     4. Disable foreign keys and perform a full load migration using
         DMS.

     5. Enable foreign keys and other constraints that may have been
         disabled during the migration

144. Migrate from Oracle database to Aurora PostgreSQL

     1. During the planning phase, the solution architect would like to
         perform an assessment of migration complexity and size, and
         identify any proprietary technology that would require
         database and application modifications. What service can help
         the solution architect with developing this assessment report
         and provide recommendations on migration strategies and
         tools?

         1. **WQF (Workload Qualification Framework)**

145. Migrate from an on-premise enterprise Oracle database to EC2
     instances

     1. To migrate an on-premise enterprise Oracle Database that
         utilizes **Oracle Data Guard** for data replication.

         1. Implement Oracle Database cluster on EC2 instances. Because
             to utilize Oracle Database Enterprise features on AWS,
             EC2 instances must be used for implementation.

         2. **RDS Option Groups** do not support Oracle Data Guard as
             an option for Oracle RDS databases.

         3. **RDS Multi-AZ** uses different technology than the
              high-availability technology provided by Oracle Data
              Guard.

     2. To migrate an enterprise Oracle database to local SSD-backed
         EC2 instances requiring high random I/O performance and high
         IOPS.

         1. **i-**family EC2 instance type provides SSD-backed instance
             storage optimized for low latency, high random I/O
             performance, and high IOPS performance.

         2. If an instance type supports both EBS backed and local
             NVMe-based SSD storage, a quick method to identify if it
             is an SSD backed EC2 type is to check if the identifier
             has a "d" in its name (e.g. "c5" vs "c5d").

146. Migrate from an on-premise enterprise Microsoft SQL Server to EC2
     instances. Current on-premise configuration utilizes SQL Server
     Reporting Services (SSRS).

     1. Install Microsoft SQL Server on EC2 instances.

     2. SQL Server Reporting Services is not available on RDS
         instances. If this service and feature is required in AWS
         cloud, the only viable option is to install Microsoft SQL
         Server database on EC2 instances. Therefore, option C is
         CORRECT and all other options are incorrect.
         ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SQLServer.html))

     3. (NEW): [RDS for SQL Server now supports SQL Server Reporting
         Services
         (SSRS)](https://aws.amazon.com/about-aws/whats-new/2020/05/amazon-rds-for-sql-server-now-supports-sql-server-reporting-services/)

147. Migrate from Informix to Aurora

     1. Manually create the **target schema** on Aurora then use **Data
         Pipeline with JDBC** to move the data.

     2. Informix is not supported by either the DMS or SCT so the only
         choice among these options is manually creating the schema.

---
## ElastiCache for Memcached vs. Redis

148. Memcached vs. Redis
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/SelectEngine.html))


149. **Caching Strategies**
     ([Ref-Memcached](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/Strategies.html),
     [Ref-Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Strategies.html),
     [Image](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/images/ElastiCache-HowECWorks.png))

     1. A **cache hit** occurs when data is in the cache and isn't
         expired.

     2. A **cache miss** occurs when data isn't in the cache or is
         expired.

     3. **Lazy loading caching strategy** loads data into the cache
         only when it is requested by the application. Thus, it
         reduces use of cache space by items not frequently requested.

         1. Use case: The application frequently requests the same
             items. The team wants to minimize cache space utilization
             and infrastructure costs.

         2. However, it updates data in cache only when a cache miss
             occurs. This means that the cache can contain stale data
             that is out of date with the data stored in the source
             database.

     4. **Write-Through caching strategy** updates the cache with every
         write operation thus ensuring that the cache always contains
         the most recent data.

         1. It is related to the consistency of data in cache in
             relation to the base source database.

         2. Use case: The application has a requirement that the data
             must always be the most recent.

150. A **node** is a fixed-size chunk of network-attached RAM. It is the
     smallest building block of an ElastiCache deployment and can run
     an instance of Memcached or Redis.

151. Reserved memory

     1. Memcached uses a metric called **SwapUsage** and if SwapUsage
         exceeds 50 MB, you need to increase the
         **memcached_connections_overhead** parameter.

     2. Redis does not use SwapUsage but instead, uses the
         **reserved-memory** metric.

152. In ElastiCache, if a primary or read replica cluster is
     unexpectedly terminated or fails, replication groups guard
     against potential data loss because your data is duplicated over
     two or more clusters. For greater reliability and faster
     recovery,

     1. create one or more read replicas in different AZs for your
         replication group, and

     2. enable Multi-AZ with auto failover on the replication group
         instead of using **Append only files (AOF)**.

153. To connect to a Memcached or Redis node, you must first connect to
     the EC2 instance using the connection utility of your choice, and
     then use the **telnet** command (Memcached) or **redis-cli**
     (Redis) to connect to the node or cluster and then run database
     commands like "set a 0 0 5" (Memcached) or "set a "hello"
     (Redis).

154. To delete a ElastiCache cluster with the CLI, you can use the
     **delete-cache-cluster** operation.

155. ElastiCache provides both host-level metrics (e.g. CPU usage) and
     metrics that are specific to the cache engine software (e.g.,
     cache gets, cache misses) in 60-second intervals.

---
## ElastiCache for Memcached

156. Max **155 TiB**, Interface: **Memcached API**, Port: **11211**

157. 1-**20 nodes per cluster** (soft limit), 100 nodes per Region (soft
     limit)

158. Availability

     1. Memcached does not support replication, so a node failure will
         always result in some data loss from your cluster.

     2. However, if you partition your data, using a greater number of
         partitions means you'll lose less data if a node fails.

     3. Also, to mitigate the impact of an AZ failure, locate your
         nodes in as many AZs as possible.

159. [NOT supported](https://aws.amazon.com/elasticache/faqs/):
     automatic failover, backup, encryption, Global Datastore,
     replication

160. Scaling cluster **horizontally** (add/remove nodes to/from a
     cluster; 1-20 nodes per cluster)
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/Scaling.html)):

     1. Changing the number of nodes in a Memcached cluster changes the
         number of the cluster's partitions.

         1. Some of your **key spaces** need to be remapped so that
             they are mapped to the correct node.

         2. Remapping key spaces temporarily increases the number of
             **cache misses** on the cluster.

     2. If you use **Auto Discovery** on your Memcached cluster, you do
         not need to change the endpoints in your application as you
         add or remove nodes.

161. Scaling cluster **vertically** (changing node type):

     1. When you scale your Memcached cluster up or down, you must
         create a new cluster.

     2. Memcached clusters always start out empty unless your
         application populates it.

162. Multi-AZ (not automatic):
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/RegionsAndAZs.html))

     1. When you create or add nodes to your Memcached cluster, you can
         specify

         1. different AZ for each node or the same AZ for all your
             nodes,

         2. allow ElastiCache to choose different AZ for each node or
             the same AZ for all your nodes.

     2. New nodes can be created in different AZs as you add them to an
         existing Memcached cluster. Once a cache node is created, its
         AZ cannot be modified.

163. Endpoints

     1. Each cache node in a Memcached cluster has its own endpoint.

     2. The cluster has an endpoint called the **cluster configuration
         endpoint**.

     3. If you enable **Auto Discovery** and connect to the
         **configuration endpoint**, your application automatically
         knows each node endpoint, even after adding or removing nodes
         from the cluster.

164. For enhanced security, ElastiCache node access is restricted to
     applications running on whitelisted EC2 instances. You can
     control the EC2 instances that can access your cluster by using
     security groups.
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/WhatIs.Components.html))

165. To connect to a Memcached node, you must first connect to the EC2
     instance using the connection utility of your choice, and then
     use the **telnet** command to connect to the node or cluster and
     then run database commands.

166. **ElastiCache eviction** occurs when a new item is added and an old
     item must be removed due to lack of free space.

     1. Scale out by adding more nodes.

     2. Scale up by increasing the memory of existing nodes.

167. To migrate Memcached nodes from a single AZ to multiple AZs

     1. Modify your cluster by creating new cache nodes in the AZs
         where you want them:

         1. --az-mode cross-az
         2. --num-cache-nodes <num of currently active cache nodes + num of new cache nodes to create>
         3. --new-availability-zones <list of AZs of the new cache nodes(optional)
         4. --apply-immediately true

     2. If you are not using **Auto Discovery**, update your client
         application with the new cache node endpoints.

     3. Modify your cluster by removing the nodes you no longer want in
         the original AZ:

         1. --nodes-to-remove <list of the cache node IDs to remove from the cluster>
         2. --num-cache-nodes <num of active cache nodes after this modification is applied>
         3. --apply-immediately true (optional)

168. In a cluster, the **max_item_size** parameter sets the size (bytes)
     of the largest item that can be stored in the cluster.

169. The default hash algorithm is **jenkins**; however, **murmur3** can
     be used if specified with the **hash_algorithm** parameter.

170. The ElastiCache Memcached Java and PHP client libraries have
     consistent hashing turned off by default, whereas it is turned on
     by default in the .Net library.

171. Upgrade: Auto OS/security patching, manual Engine Version Upgrade

---
## ElastiCache for Redis

172. Redis is a key-value store that supports abstract data types and
     optional data durability.

     1. Redis can offer not only caching functionality but also
         **Pub/Sub**, **Sorted Sets** and an **In-Memory Data Store**.

     2. Redis provides native capability for **sorted sets** which is
         an ideal solution for **complex and compute intensive data
         structures** (e.g. leaderboards).

     3. ElastiCahe can improve latency and throughput for **read-heavy
         applications**.

173. Max **155.17 TiB of in-memory key-value store**, Interface: **Redis
     API**, Port: **6379**

174. Redis **cluster** (called **replication group** in the API/CLI),

175. **Shard** (called **node group** in the API/CLI)

176. **Each shard = max 6 nodes (1 primary + max 5 read replicas)**. If
     you have no replicas and a node fails, you experience loss of all
     data in that shard.

177. Each **node** in **shard** has the same compute, storage, and
     memory specifications.

178. ElastiCache uses DNS entries to allow client applications to locate
     cache servers (Cache Nodes).

     1. Each cache **node** has its own DNS name and port.

     2. The DNS name for a Cache Node remains constant, but the IP
         address of a Cache Node can change over time (e.g. when Cache
         Nodes are auto replaced after a failure).

179. **Redis (cluster mode disabled)** vs. **Redis (cluster mode
     enabled)**

     1. **Redis (cluster mode disabled)** can add or delete nodes from
         the cluster.

     2. **Redis (cluster mode enabled)** clusters: the structure of the
         cluster, node type, number of shards, and number of nodes, is
         fixed at the time of creation and cannot be changed.

     3. Scaling vs. partitioning
         ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/nodes-select-size.html#CacheNodes.SelectSize))

         1. **Redis (cluster mode disabled)** supports **scaling**. You
             can scale read capacity by adding or deleting replica
             nodes or scale capacity by scaling up to a larger node
             type.

         2. **Redis (cluster mode enabled)** is better for
             **partitioning** data across shards, not scaling.

             1.  Allow dynamically change the number of shards as your
                 business needs change.

             2.  Spread load over a greater number of endpoints, reduce
                 access bottlenecks during peak.

             3.  Accommodate a larger data set since the data can be
                 spread across multiple servers.

     4. Reads vs. writes
         ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.Redis-RedisCluster.html))

         1. For **read-heavy** applications, it is preferable to use
             **Redis (cluster mode disabled)** clusters that support
             scaling up or down of read capacity by creating and
             deleting read replicas within the cluster. However, there
             is a maximum of **5 read replicas**.

             1.  Horizontally scaling **Redis (cluster mode enabled)**
                 clusters can be a complex activity as it can require
                 re-sharding of the clusters.

         2. For **write-heavy** applications, it is preferable to use
             **Redis (cluster mode enabled)** cluster because it
             provides multiple **write-endpoints** (multiple shards)
             which can be used to distribute traffic for write-heavy
             applications.

     5. Node type vs. number of nodes

         1. Since **Redis (cluster mode disabled) cluster** has only
             **one** shard, the **node type** must be large enough to
             accommodate all the cluster's data plus necessary
             overhead.

         2. When using a **Redis (cluster mode enabled) cluster,**
             since you can partition your data across several shards,
             the node types can be smaller, though you need more of
             them.

         3. ElastiCache does NOT currently support dynamically
              changing the cache node type for a cache cluster after
              it has been created. If you wish to change the Node Type
              of a cache cluster, you will need to set up a new cache
              cluster with the desired Node Type, and migrate your
              application to that cache cluster.

180. **Redis (cluster mode disabled) cluster**
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.Redis-RedisCluster.html),
     [Scaling](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/scaling-redis-classic.html))

     1. **1 shard**; with **0-5 read replica nodes** for each shard

     2. Data partitioning NOT supported.

     3. Promote replica to primary: not automatic

     4. Multi-AZ: optional

     5. For **single-node cluster** with 0 shards (use the one node for both reads and writes)
         ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Clusters.html))

        1. To change **data capacity** of a cluster, scale up/down a
             single-node Redis **cache cluster**

                aws elasticache **modify-cache-cluster**
                    **--cache-cluster-id** my-redis-cache-cluster
                    --cache-node-type cache.m3.xlarge
                    --cache-parameter-group-name redis32-m2-xl
                    --apply-immediately

     6. For **multi-node clusters** with 1 shard (1 node as the read/write primary node with **0-5 read replica nodes**).

        1. To change **data capacity** of a cluster, scale up/down a Redis
            **replication group (cluster)**

                aws elasticache **modify-replication-group**
                    **--replication-group-id** my-repl-group
                    --cache-node-type cache.m3.xlarge
                    --cache-parameter-group-name redis32-m2-xl
                    --apply-immediately

        2. To change the **read capacity** of a cluster, add more read replicas
            (max 5), or remove read replicas.

181. **Redis (cluster mode enabled) cluster**
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.Redis-RedisCluster.html),
     [Shards](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Shards.html))

     1. **1- 500 shards per cluster** if the Redis engine version is 5.0.6 or higher
     2. **1- 250 shards per cluster** for Redis versions below 5.0.6
     3. **0-5 read replica nodes** for each shard, **max 90 nodes** per cluster (soft limit can increase to 500)
     4. Promote replica to primary: automatic
     5. Multi-AZ: required
     6. **Horizontal scaling** (change the **number of shards** in the cluster)

         1. **Online resharding** and **shard rebalancing**, allows
             scaling in/out while the cluster continues serving
             incoming requests with no downtime.

         2. **Offline resharding** and **shard rebalancing,** in
             addition to changing the number of shards in your
             replication group, you can do the following:

             1.  Change the node type of your cluster.
             2.  Specify the AZ for each node in the cluster.
             3.  Upgrade to a newer engine version.
             4.  Specify the number of replica nodes in each shard
                 independently.
             5.  Specify the keyspace for each shard.

     7. **Online vertical scaling**, which changes the **node type** to
         resize the cluster, allows scaling up/down while the cluster
         continues serving incoming requests.

     8. **Max 90 nodes per cluster** (soft limit). The node or shard
         limit can be increased to a max of **500 per cluster**.

         1. E.g. If each shard is planned to have 5 replicas, max no.
             of shards = 90 / (5+1) = 15 shards.

         2. E.g. If each shard is planned to have 5 replicas, max no.
             of shards = 500 / (5+1) = 83 shards
             ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Shards.html))

     1. Make sure there are enough available IP addresses to
         accommodate the increase. Common pitfalls include the subnets
         in the subnet group have too small a CIDR range or the
         subnets are shared and heavily used by other clusters.
         ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Shards.html))

182. Endpoints
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Endpoints.html))

     1. **Redis standalone node**, use the node's endpoint for both
         read and write operations.

     2. **Redis (cluster mode disabled) clusters**

         1. **Primary Endpoint** is for all write operations.

         2. **Reader Endpoint** evenly splits incoming connections to
             the endpoint between all read replicas. A **reader
             endpoint** is not a load balancer. It is a DNS record
             that will resolve to an IP address of one of the replica
             nodes in a round robin fashion.

         3. Use the individual **Node Endpoints** for read operations
              (aka **Read Endpoints** In API/CLI).

     3. **Redis (cluster mode enabled) clusters**

         1. The cluster's **Configuration Endpoint** is for all
             operations.

         2. You can still read from individual **Node Endpoints** (aka.
             Read Endpoints in API/CLI).

183. Connecting to a Cluster's Node
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/GettingStarted.ConnectToCacheNode.htmlhttps://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/GettingStarted.ConnectToCacheNode.html))

     1. Use **redis-cli** to connect to a Redis cluster that is not
         encryption-enabled. You can then run database commands.

184. Multi-AZ and Automatic failover: optional
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Replication.Redis-RedisCluster.html),
     [FAQs](https://aws.amazon.com/elasticache/redis/faqs/))

     1. You can use Multi-AZ if you have a cluster consisting of a
         primary node and one or more read replicas.

     2. Multi-AZ is enabled by default on **Redis (cluster mode
         enabled) clusters**, if the cluster contains at least one
         replica in a different AZ from the primary in all shards.

     3. If Multi-AZ is disabled, ElastiCache replaces a cluster's
         failed node by recreating/reprovisioning the failed node.

     4. If Multi-AZ is enabled, a failed primary node fails over to the
         replica with **the least replication lag**.

         1. The selected replica is automatically promoted to primary,
             which is much faster than creating and reprovisioning a
             new primary node. This process usually takes just **a few
             seconds** until you can write to the cluster again.

         2. A replacement read replica is then created and provisioned
             in the same AZ as the failed primary.

         3. In case the primary failed due to temporary AZ disruption,
              the new replica will be launched once that AZ has
              recovered.

         4. **Primary endpoint** -- You don't need to make any changes
             to your application, because the DNS name of the new
             primary node is propagated to the primary endpoint.

         5.  **Read endpoint** -- The reader endpoint is automatically
             updated to point to the new replica nodes.

     5. ElastiCache **--automatic-failover-enabled** flag is used for
         multi-AZ automatic failover.

     6. When you manually promote read replicas to primary on **Redis
         (cluster mode disabled)**, you can do so only when **Multi-AZ
         and automatic failover are disabled**.
         ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/AutoFailover.html))

     7. You cannot disable Multi-AZ on **Redis (cluster mode enabled)**
         clusters. Therefore, you cannot manually promote a replica to
         primary on any Redis (cluster mode enabled) cluster.

     8. Whenever the primary is rebooted, it is cleared of data when it
         comes back online, which in turn, results in the replicas
         clearing their copy of the data. This results in data loss.

185. Automatic Backups and Manual Backups

     1. For persistence, Redis supports **point-in-time** backups
         (copying the Redis data set to disk).
         ([Ref](https://aws.amazon.com/redis/))

     2. Automatic Backups of the cluster (daily) are retained in
         **S3**. Backup retention period: **0 - 35 days**

     3. During the backup process, you cannot run any other API or CLI
         operations on the cluster.

     4. During any contiguous **24-hour** period**,** you can create no
         more than **20 manual backups** per node in the cluster.

     5. You can export a backup to a S3 bucket so you can access it
         from outside ElastiCache.
         ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/backups-exporting.html))

         1. Exporting a backup can be helpful if you need to launch a
             cluster in another Region.

         2. The ElastiCache backup and the S3 bucket that you want to
             copy it to must be in the same Region.

     6. To create a final backup of an ElastiCache Redis cluster
         ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/backups-final.html))

         1. aws elasticache delete-cache-cluster --cache-cluster-id
             myCluster --final-snapshot-identifier xxx

     7. If you delete a cluster and request a final backup, ElastiCache
         always takes the backup from the **primary node**. This
         ensures that you capture the very latest Redis data, before
         the cluster is deleted.

     8. **Redis (cluster mode disabled) cluster**

         1. Backups (Snapshots): A single .rdb file.

         2. Restore data to a new cluster using a single .rdb file from
             a Redis (cluster mode disabled) cluster.

         3. Backup and restore aren't supported on cache.t1.micro
              nodes. All other cache node types are supported.

     1. **Redis (cluster mode enabled) cluster**

         1. Backups (Snapshots): A **.rdb file for each shard**.

         2. Restore data to a new cluster .rdb files from either a
             **cluster mode disabled** or **cluster mode enabled**
             cluster.

         3. Support taking backups on the **cluster level**; *NOT* at
              the **shard level**.

     10. When seeding a new ElastiCache Redis cluster from a backup
         file, if the backup does not fit in the memory of the new
         node, the resulting cluster will have a status of
         **restore-failed**. If this happens, you must delete the
         cluster and start over.

     11. An **ElastiCache:SnapshotFailed** event means that ElastiCache
         was unable to populate the cache cluster with Redis snapshot
         data.

186. If you experience performance issues occur during the ElastiCache
     automated backup window, (4)

     1. Create backups from one of the read replicas.

     2. Set the **reserved-memory-percentage** parameter (to 25%
         recommended), which specifies the amount of memory available
         for non-data use (background processes).

         1. If all of a node's available memory is consumed, then
             excessive paging to the disk can occur.

187. If you experience performance issues because the ElastiCache
     cluster does not have sufficient memory allocated for non-data
     use, create a **custom parameter group** with
     **reserved-memory-percent** parameter to 50. Apply the **custom
     parameter group** to the cluster.

     1. It is
         [recommended](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/redis-memory-management.html)
         to use the **reserved-memory-percent** parameter (not
         **reserved-memory**), because each node in a cluster can be
         of different types, each node can have a different maximum
         memory.

188. A large and sustained spike in the number of concurrent connections
     indicates there has been either

     1. a large, sustained traffic spike or

     2. the application is not releasing connections as it should.

189. In a Redis ElastiCache cluster, the **timeout** parameter can be
     set in order for Redis to disconnect idle clients.

190. Multi-region: **Global Datastore** feature can be used to create
     **cross-region read replica** clusters.

191. **Global Datastores**
     ([Ref](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/Redis-Global-Datastore.html),
     [Image](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/images/Global-DataStore.png))

     1. Each global datastore is a collection of one or more clusters
         that replicate to one another.

         1. The **Primary (active) cluster** accepts writes that are
             replicated to all clusters within the global datastore. A
             primary cluster also accepts read requests.

         2. A **Secondary (passive) cluster** only accepts read
             requests and replicates data updates from a primary
             cluster. A secondary cluster needs to be in a different
             Region than the primary cluster.

     2. ElastiCache manages automatic, **asynchronous replication** of
         data between the two clusters.

     3. Advantages

         1. Geolocal performance -- you can reduce latency of data
             access in that Region.

         2. Disaster recovery -- If your primary cluster in a global
             datastore experiences degradation, you can promote a
             secondary cluster as your new primary cluster
             (**manually)**.

     4. ElastiCache **does NOT support autofailover** from one Region
         to another. When needed, you can promote a secondary cluster
         as the new primary cluster manually.

     5. All primary and secondary clusters in your global datastore
         should have the same number of primary nodes, node type,
         engine version, and number of shards (in case of
         **cluster-mode enabled**).

     6. Each cluster in your global datastore can have a **different
         number of read replicas** to accommodate the read traffic
         local to that cluster.

     7. You can scale regional clusters both vertically (scaling up and
         down) and horizontally (scaling in and out). You can scale
         the clusters by modifying the global datastore. All the
         regional clusters in the global datastore are then scaled
         **without** interruption.

     8. **Replication must be enabled** if you plan to use an existing
         single-node cluster.

     1. You can work with global datastores only in VPC clusters.

     10. Global datastores support **pub/sub messaging** with the
         following stipulations:

         1. For **cluster-mode disabled**, pub/sub is fully supported.
             Events published on the primary cluster of the primary
             Region are propagated to secondary Regions.

         2. For **cluster mode enabled**, the following applies:

             1.  For published events that aren't in a key space, only
                 subscribers in the same Region receive the events.

             2.  For published key space events, subscribers in all
                 Regions receive the events.

192. To cancel a subscription to a single Redis pub/sub channel, you
     should use the UNSUBSCRIBE command specifying the channel you
     want to cancel.

193. Encryption: optional, applies to disk during sync, backup and swap
     operations, backups stored in S3.

194. Access Control and server authentication

     1. Use Redis AUTH which gives all users the same permissions if
         the Token provided is a match,

     2. Use the **User Group Access Control List** as the Access
         Control Option. This allows you to authenticate users on a
         Role Based Access Control method (RBAC).

195. Upgrade: Auto OS/security patching, manual Engine Version Upgrade

196. If you create an IAM role in your AWS account with permissions to
     create a cache cluster, and someone who assumes the role creates
     a cluster, then who will own the cache cluster resource?

     1. Your AWS account

197. **add-tags-to-resource** can be used to add or modify the value of
     a tag from an existing ElastiCache resource.

---
## Redshift

198. Redshift is a **peta-byte** scale **relational columnar datastore**
     used as a data warehouse and optimized for reporting and business
     intelligence applications.

     1. Redshift databases are designed as analytical repositories.
         They can store aggregate values from transactional databases
         and dozens of other source locations.

     2. Redshift databases are compatible with all data types, although
         semistructured and unstructured data may require
         preprocessing before it can be loaded into the data
         warehouse.

     3. Redshift is not suitable for ingestion of data at low latency.

     4. Data warehouses are not used to store highly detailed records.

199. Max **8 PB**, Interface **SQL**, Port (TCP) **5439**

200. Upgrade: Auto **Cluster version** upgrade (periodically), version
     [options](https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-clusters.html#rs-mgmt-maintenance-tracks):
     **Current, Trailing, Preview**

201. Single-AZ: ([Ref](https://aws.amazon.com/redshift/faqs/))

     1. Redshift only supports Single-AZ deployments.

     2. You can run data warehouse clusters in multiple AZ's by
         loading data into **two Redshift data warehouse clusters** in
         separate AZs from the same set of S3 input files.

     3. You can restore a data warehouse cluster to a different AZ from
         your **data warehouse cluster snapshots**.

     4. With **Redshift Spectrum**, you can spin up multiple clusters
         across AZs and access data in S3 without having to load it
         into your cluster.

202. No Failover. In the event of individual node failure:

     1. Redshift will automatically detect and replace a failed node in
         your data warehouse cluster.

     2. The data warehouse cluster will be unavailable for queries and
         updates until a replacement node is provisioned and added to
         the DB.

     3. Redshift makes your replacement node available immediately and
         loads your most frequently accessed data from S3 first to
         allow you to resume querying your data as quickly as
         possible.

     4. Single node clusters do not support data replication. Recommend
         using at least 2 nodes for production.

     5. In the event of a **drive failure**, you will need to restore
         the cluster from snapshot on S3.

203. If AZ has an outage, Redshift will automatically move your cluster
     to another AZ without any data loss or application changes. To
     activate this, enable the **relocation capability** in your
     cluster configuration settings.

204. Disaster recovery (in case of one of the Region fails),

     1. Configure Redshift to automatically copy snapshots (either
         automated or manual) for a cluster to another region.

         1. When a snapshot is created in the cluster's primary region,
             it is copied to a secondary region; these are known as
             the source region and destination region.

         2. Then you have the ability to restore your cluster from
             recent data if anything affects the primary region.

205. Scaling (changing the node type and number of nodes)
     ([Ref](https://docs.aws.amazon.com/redshift/latest/mgmt/managing-cluster-operations.html))

     1. **Elastic resize**

         1. Use elastic resize to change the node type, number of
             nodes, or both.

         2. If you only change the number of nodes, then queries are
             temporarily paused and connections are held open if
             possible.

         3. During the resize operation, the cluster is read-only.

         4. Typically, elastic resize takes **10-15 mins**.

     2. **Classic resize**

         1. Use classic resize to change the node type, number of
             nodes, or both.

         2. Choose this option when you are resizing to a configuration
             that isn't available through elastic resize.

             1.  E.g. **to or from a single-node cluster**.

         3. During the resize operation, the cluster is read-only.

         4. Typically, classic resize takes **2 hours - 2 days or
             longer**, depending on your data's size.

     3. **Snapshot and restore with classic resize**

         1. To keep your cluster available during a classic resize, you
             can first make a copy of an existing cluster, then resize
             the new cluster.

206. Encryption: **KMS** or **HSM** or **None**, applies to cluster's
     storage volume, data, indexes, logs, automated backups, and
     snapshots.

207. Automatic Backups and Manual Snapshots

     1. Automatic Backup retention period: **0 - 35 days (default 1
         day)**. If you disable automated snapshots, Redshift stops
         taking snapshots and deletes any existing automated snapshots
         for the cluster.

     2. **Redshift snapshots** are stored on **S3**.

     3. The default retention period for **copied snapshots** is **7
         days** and it only applies to **automated snapshots**.

     4. Redshift always attempts to maintain **at least 3 copies of
         your data** (the original and replica on the compute nodes,
         and a backup in S3).

     5. Redshift can also **asynchronously** replicate your snapshots
         to **S3 in another region** for disaster recovery.

     6. The automated backup and manual cluster snapshots include

         1. Public tables

         2. Labels

         3. Replica tables

         4. BUT NOT System tables

     7. To copy an Redshift cluster from a production AWS account to a
         non-production AWS account.

         1. Create a manual snapshot of the source Redshift cluster.
             Share the snapshot with the target AWS account. In the
             target AWS account, create a new Redshift cluster by
             restoring from the shared snapshot.
             ([Ref](https://aws.amazon.com/premiumsupport/knowledge-center/account-transfer-redshift/))

     8. To copy encrypted RedShift snapshots to another region

         1. If working from the CLI, create a snapshot copy grant in
             the destination Region for a KMS key in the destination
             Region. Configure Redshift cross-Region snapshots in the
             source Region.

         2. If working from the console, Redshift provides the proper
             workflow to configure the grant when you enable
             cross-Region snapshot copy.

208. Audit logging is not enabled by default in RedShift. If logging is
     enabled, the logs are stored in S3.

209. **Redshift clusters** are composed of **nodes**. **Compute nodes**
     divide work among **slices**. Each **slice** is assigned a
     portion of the node's memory and drive space. When you connect to
     a Redshift cluster, you use the **SQL endpoint**.

210. **Single-node cluster** vs. **Multi-node cluster**

     1. **Single-node cluster** enables you to get started with
         Redshift quickly and cost-effectively and scale up to a
         multi-node configuration as your needs grow.

     2. **Multi-node cluster** requires a leader node that manages
         client connections and receives queries, and **two compute
         nodes** that store data and perform queries and computations.
         The leader node is provisioned for you automatically and you
         are not charged for it.

     3. **Redshift single-node** does not support replication.

     4. **Redshift multi-node clusters** support node recovery.

211. **Redshift Spectrum** can be used to efficiently query and retrieve
     structured and semistructured data from files in S3 without
     having to load the data into Redshift tables.

     1. Redshift Spectrum queries employ massive parallelism to execute
         very fast against large datasets. Much of the processing
         occurs in the Redshift Spectrum layer, and most of the data
         remains in S3.

212. **Connecting to the Leader node**

     1. The leader node is the only node in your cluster you can
         directly connect to.

     2. The SQL endpoints provided by AWS are the DNS of the leader
         node.

     3. Typically you connect using a SQL client and a JDBC or ODBC
         driver.

     4. **SQL Clients:** Postico, MySQL Workbench, Command Line SQL
         tools

     5. **SQL Drivers**: JDBC, ODBC

     6. **SQL Queries**: not exactly the same as PostgreSQL

213. **Compute Node Types**

     1. **Dense compute** (**dc**) nodes provide less storage space but
         improved performance with SSDs and higher IO.

     2. **Dense storage** (**ds**) nodes provide significantly higher
         amounts of storage but less performance.

     3. Max of **128 nodes** of the large sizes.

214. Security

     1. Redshift is secured at the cluster level, not the node level.

     2. You can encrypt data using keys you manage through KMS.

     3. Connections to the database are secured using HTTPS.

     4. IAM is used to authorize and authenticate users to Redshift
         clusters.

     5. Redshift clusters are run inside of a **VPC**. You can use a
         VPN to connect your on-premises data center to Redshift, but
         it does not contain or isolate a Redshift cluster.

         1. Security Group Inbound Rule: Type="Redshift (5439)",
             Protocol="TCP (6)"

         2. You cannot change the port number for your Redshift cluster
             after it is created

215. Load data

     1. To validate the data in the S3 input files or DynamoDB table
         before you actually load the data in Redshift, use the
         **NOLOAD option** with the **COPY command**.

     2. You can load data from EMR using the

         1. Hadoop Distributed File System (HDFS), or

         2. Custom bootstrap action.

216. If a scheduled maintenance occurs while a query is running, the
     query is terminated and rolled back and you need to restart it.
     ([Ref](https://docs.aws.amazon.com/redshift/latest/dg/c_best-practices-avoid-maintenance.html))

     1. To minimize data load problems caused by a maintenance window

         1. Schedule **VACUUM** operations to avoid future maintenance
             windows

         2. Re-execute queries or transactions that were terminated and
             rolled back

217. **S3ServiceException** Errors
     ([Source-1](https://docs.aws.amazon.com/redshift/latest/dg/t_Troubleshooting_load_errors.html))

     1. caused by an improperly formatted or incorrect credentials
         string, having your cluster and your bucket in different
         regions, and insufficient S3 privileges.

218. System Tables for troubleshooting data loads
     ([Source-1](https://docs.aws.amazon.com/redshift/latest/dg/t_Troubleshooting_load_errors.html))

     1. Query **STL_LOAD_ERRORS** to discover the errors that occurred
         during specific loads.

     2. Query **STL_FILE_SCAN** to view load times for specific files
         or to see if a specific file was even read.

     3. Query **STL_S3CLIENT_ERROR** to find details for errors
         encountered while transferring data from S3.

219. Multibyte Character Load Errors
     ([Source-1](https://docs.aws.amazon.com/redshift/latest/dg/t_Troubleshooting_load_errors.html))

220. You want to test the performance of your Redshift cluster and see
     the baseline of some queries on your tables. You make a new query
     and test it. The first query executes in 300s. You rerun the same
     query, which executes in 150s. What would explain the difference
     in the execution time?

     1. The first query includes the compilation time.

     2. In Redshift, when you compare query execution times, do not
         count the first time the query is executed, because the first
         run time includes the compilation time.

221. You've noticed that your VPN connection to Redshift appears to hang
     or timeout when running long queries, such as a COPY command. You
     observe that the Redshift console displays the completion of the
     query, but the client tool itself still appears to be running the
     query and results are incomplete. What are some ways to
     troubleshoot this problem?

     1. This happens when you connect to Redshift from a computer other
         than an EC2 instance, and idle connections are terminated by
         an intermediate network component, such as a firewall, after
         a period of inactivity. This behavior is typical when you log
         in from a VPN or your local network.

     2. To avoid these timeouts, Redshift recommends these changes:

         1. Increase client system values that deal with TCP/IP
             timeouts. You should make these changes on the computer
             you are using to connect to your cluster. The timeout
             period should be adjusted for your client and network.

         2. Optionally, set keep-alive behavior at the DNS level.

222. You have a Lambda function that creates the same number of records
     in both DynamoDB and Redshift. You checked the number of their
     concurrent executions and found that the concurrent executions
     count does not match.

     1. One record in Redshift equates to two records in DynamoDB
         Stream.

     2. Concurrent executions refers to the number of executions of
         your function code that are happening at any given time. You
         can estimate the concurrent execution count, but the
         concurrent execution count will differ depending on whether
         or not your Lambda function is processing events from a
         stream-based event source. Since DynamoDB stream is
         stream-based and Redshift is not, it's normal for the
         concurrent execution count to differ.

**More Redshift Details**

223. Redshift performance data (both CloudWatch metrics and query and
     load data) is recorded **every minute**.

224. Billing: **Concurrency Scaling** and **Redshift Spectrum** are
     optional features of Redshift that you will be billed for if
     enabled.

225. In Redshift, the **masteruser**, which is the user you created when
     you launched the cluster, is a **superuser**. Database superusers
     have the same privileges as database owners for all databases.
     You must be a superuser to create a superuser. To create a new
     database superuser, log on to the database as a superuser and
     issue a CREATE USER command or an ALTER USER command with the
     CREATEUSER privilege.

226. To make a table read-only for everyone including the object owners

     1. The object owners can revoke their own ordinary privileges
         using the **REVOKE** command and then make the table
         read-only for themselves as well as others.

     2. The right to modify or destroy an object is always the
         privilege of the owner only.

     3. The privileges of the object owner, such as DROP, GRANT, and
         REVOKE privileges, are implicit and cannot be granted or
         revoked.

     4. Object owners can revoke their own ordinary privileges, for
         example, to make a table read-only for themselves as well as
         others. Superusers retain all privileges regardless of GRANT
         and REVOKE commands.

227. Performance Tuning

     1. Redshift can run queries across **petabytes** of data in
         Redshift itself and **exabytes** of data in S3.

     2. Redshift can achieve its massive speed improvements by
         implementing **columnar indexing** of the data and **parallel
         processing**.

     3. When you create tables in Redshift

         1. Choosing the best sort key
         2. Choosing the best distribution key
         3. Choosing the best compression strategy
         4. Defining constraints

     4. **Differences and Optimizations**

         1. Uniqueness, primary key, and foreign key constraints are
             informational only; they are not enforced by Redshift.

         2. By default, Redshift stores data in its raw, uncompressed
             format. You can apply a compression type, or encoding, to
             the columns in a table manually when you create the
             table, or you can use the COPY command to analyze and
             apply compression automatically.

         3. **Data compression** allows for significant performance
              improvements.

              1.  Different compression methods are offered when
                  creating tables.

              2.  Tools are provided to determine the most effective
                  compression algorithms.

              3.  Though users can specify encodings for compression,
                  **automatic compression** produces the best results.

              4.  **Delta encoding** is very useful for datetime
                  columns.

         4. **Columnar storage**

             1.  Data is stored column-by-column, not row-by-row.

             2.  Give significant performance gain if just interact with
                 data from one column.

             3.  Columnar storage allows for more effective compression.

         5.  **Zone Maps**

             1.  A chunk of data with metadata that can be evaluated to
                 determine if a query even needs to interact with the
                 chunk to execute.

             2.  In-memory MIN and MAX values to optimize queries and
                 prune blocks that cannot contain data.

         6.  Zonal deployment

             1.  Not deployed in multiple AZs (by default) in order to
                 reduce latency.

         7.  Redshift stores DATE and TIMESTAMP data more efficiently
              than CHAR or VARCHAR, which results in better query
              performance.

         8. Designed for large writes

               1.  Batch processing and high parallelism means Redshift
                   is optimized for large amounts of data.

               2.  Small writes of 1-100 rows have similar cost to
                   larger writes of ~100k rows.

     5. To clean up tables after a bulk delete, a load, or a series of
         incremental updates, you need to run the VACUUM command,
         either against the entire database or against individual
         tables.

         1. You can run a VACUUM FULL, VACUUM DELETE ONLY, or VACUUM
             SORT ONLY.

228. When you create a table in Redshift, you designate one of three
     distribution styles; **EVEN**, **KEY**, or **ALL**.

     1. After you have specified a distribution style for a column, Redshift
        handles data distribution at the cluster level.

229. Redshift provides access to the following types of system tables:

     1. STL Tables for Logging
     2. STV Tables for Snapshot Data
     3. System Views
     4. System Catalog Tables

---
## DocumentDB

231. Max **64 TB per database cluster**, Interface: **subset of MongoDB API**

232. Database port: **27102**

233. A cluster consists of 0 - 16 instances.

     1. One **primary DB instance**, supports read/write
         operations**,** performs all the data modifications to the
         **cluster volume**.

     2. A **cluster volume** is where data is stored. It is a single,
         virtual volume that uses solid-state disk (SSD) drives and
         designed for reliability and high availability. The cluster
         volume consists of copies of the data across 3 AZs in a
         single Region.

     3. **0-15 Read replicas**, support only **read**
         operations,connect to the same storage volume as the primary
         instance.

234. **Endpoints of a cluster**

     1. One **cluster endpoint**, connects to the current primary DB
         instance for the DB cluster.

         1. Example endpoint:
             sample-cluster.cluster-123456789012.us-east-1.docdb.amazonaws.com:27012

         2. The cluster endpoint can be used for read and write
             operations.

         3. The cluster endpoint provides failover support for read
              and write connections to the cluster. If your cluster's
              current primary instance fails, and your cluster has at
              least one active read replica, the cluster endpoint
              automatically redirects connection requests to a new
              primary instance.

     2. One **reader endpoint** provides **automatic load-balancing**
         support for read-only connections to the DB cluster.

         1. Example endpoint:
             sample-cluster.cluster-ro-123456789012.us-east-1.docdb.amazonaws.com:27012

         2. Each Aurora DB cluster has one reader endpoint.

         3. If the cluster only contains a primary instance and no
              read replicas, the reader endpoint connects to the
              primary instance.

         4. When you add a replica instance to a cluster, the reader
             endpoint opens read-only connections to the new replica
             after it is active.

         5.  The reader endpoint load balances read-only connections,
             not read requests. If some reader endpoint connections
             are more heavily used than others, your read requests
             might not be equally balanced among instances in the
             cluster. It is recommended to distribute requests by
             connecting to the cluster endpoint as a **replica set**
             and utilizing the **secondaryPreferred read preference
             option**.

         6.  Attempting to perform a write operation over a connection
             to the reader endpoint results in an error.

     3. An **instance endpoint** per each instance, connects to a
         specific DB instance.

         1. Example endpoint:
             sample-instance.123456789012.us-east-1.docdb.amazonaws.com:27012

         2. The instance endpoint provides direct control over
             connections to the DB cluster, for scenarios where using
             the cluster endpoint or reader endpoint might not be
             appropriate.

         3. The instance endpoint for the current primary instance can
              be used for read and write operations.

         4. Attempting to perform write operations to an instance
             endpoint for a read replica results in an error.

         5.  Use case: Provisioning for a periodic read-only analytics
             workload.

                You can provision a larger-than-normal replica instance, connect
                directly to the new larger instance with its instance endpoint, run
                the analytics queries, and then terminate the instance. Using the
                instance endpoint keeps the analytics traffic from impacting other
                cluster instances.

     4. **Replica Set Mode**

        1. You can connect to your DocumentDB cluster endpoint in **replica
            set mode** by specifying the replica set name **rs0**.

        2. Connecting in replica set mode provides the ability to specify
            the options:
            ([read-consistency](https://docs.aws.amazon.com/documentdb/latest/developerguide/how-it-works.html#durability-consistency-isolation.read-consistency))

            1.  **Read Concern**

            2.  **Write Concern**, and

            3.  **Read Preference** .

        3. When you connect in replica set mode, your DocumentDB cluster
                appears to your drivers and clients as a replica set.
                Instances added and removed from your DocumentDB cluster are
                reflected automatically in the replica set configuration.

        4. Each DocumentDB cluster consists of a single replica set with
            the default name **rs0**. The replica set name cannot be
            modified.

        5.  Connecting to the cluster endpoint in replica set mode is the
            recommended method for general use.

        6.  All instances in a DocumentDB cluster listen on the same TCP
            port for connections.

        7.  Example connection string:
                mongodb://username:password@sample-cluster.cluster-123456789012.us-east-1.docdb.amazonaws.com:27017/?replicaSet=rs0

235. Scaling - Storage
     ([Ref](https://docs.aws.amazon.com/documentdb/latest/developerguide/db-cluster-manage-performance.html),
     [high water
     mark](https://docs.aws.amazon.com/documentdb/latest/developerguide/how-it-works.html))

     1. Storage will automatically grow, up to **64 TB**, in 10GB
         increments with no impact to database performance. The size
         of your cluster volume is checked on an hourly basis.

     2. Scaling down is NOT supported. Storage costs are based on the
         storage "high water mark" (the maximum amount allocated to
         your DocumentDB cluster at any time during its existence)

         1. Avoid ETL practices that create large amounts of temporary
             information, or that load large amounts of new data prior
             to removing unneeded older data.

         2. You can determine what the "high water mark" is currently
             for your DocumentDB cluster by monitoring the
             **VolumeBytesUsed** CloudWatch metric.

         3. If removing data from a DocumentDB cluster results in a
              substantial amount of allocated but unused space,
              resetting the high water mark requires doing a **logical
              data dump** and restore to a new cluster, using a tool
              such as **mongodump** or **mongorestore**.

         4. Creating and restoring a snapshot does NOT reduce the
             amount allocated storage, because the physical layout of
             the underlying storage remains unchanged.

236. Scaling - Instance Scaling

     1. Scale your DocumentDB cluster as needed by modifying the
         instance class for each instance in the cluster. DocumentDB
         supports several instance classes that are optimized for
         DocumentDB.

     2. **db.r4.large** is the smallest instance class for DocumentDB
         instances.

     3. Set another alarm for replication lags that exceed 10s. If you
         surpass this threshold for multiple data points, scale up
         your instances or reduce your write throughput on the primary
         instance.

237. Scaling - Read Scaling

     1. Create up to **15 replicas** in the DB cluster. Read replicas
         don't have to be of the same DB instance class as the
         primary instance.

     2. DocumentDB replica returns with minimal replication lag less
         than **100 milliseconds**.

     3. To read scale with DocumentDB, we recommend that you connect to
         your cluster as a **replica set** and distribute reads to
         replica instances using the **built-in read preference
         capabilities of your driver**.

238. Scaling - Write Scaling (manual)

     1. Add a replica of a larger instance type to your cluster.

     2. Set the **failover tier** on the new replica to priority zero,
         ensuring a replica of the smaller instance type has the
         highest failover priority.

     3. Initiate a manual failover to promote the new replica to be the
         primary instance.

     4. This will incur ~30 seconds of downtime for your cluster.
         Please plan accordingly.

     5. Remove all replicas of an instance type smaller than your new
         primary from the cluster.

     6. Set the failover tier of all instances back to the same
         priority (usually, this means setting them back to 1)

239. Multi-AZ and automatic failover
     ([Ref](https://docs.aws.amazon.com/documentdb/latest/developerguide/replication.html),
     [failover](https://docs.aws.amazon.com/documentdb/latest/developerguide/failover.html))

     1. DocumentDB features fault-tolerant, self-healing storage that
         replicates six copies of your data across 3 AZs.

     2. In a DocumentDB cluster, DocumentDB provisions and
         automatically distributes instances across the AZs in the
         subnet group (must be **at least 2 AZs**) to balance the
         cluster (prevents all instances from being located in the
         same AZ).

     3. If you have a DocumentDB replica instance in the same or
         different AZ when failing over: DocumentDB flips the CNAME
         for your instance to point at the healthy replica, which is,
         in turn, promoted to become the new primary. Failover
         typically completes within **30 seconds** from start to
         finish.

     4. If you don't have a DocumentDB replica instance (for example,
         a single instance cluster): DocumentDB will attempt to create
         a new instance in the same AZ as the original instance. This
         replacement of the original instance is done on a best-effort
         basis and may not succeed if, for example, there is an issue
         that is broadly affecting the AZ.

     5. To force a failover, use the failover-db-cluster operation with
         these parameters.

         1. --db-cluster-identifier <cluster-name>

         2. --target-db-instance-identifier <name of the instance to
             be promoted to primary(optional)

     6. DocumentDB provides you with **failover tiers** as a means to
         control which replica instance is promoted to primary when a
         failover occurs.

     7. Each replica instance is associated with a failover tier
         **(0--15)**. When a failover occurs due to maintenance or an
         unlikely hardware failure, the primary instance fails over to
         a replica with the lowest numbered priority tier. If multiple
         replicas have the same priority tier, the primary fails over
         to that tier's replica that is the closest in size to the
         primary.

     8. By setting the failover tier for a group of select replicas to
         0 (the highest priority), you can ensure that a failover will
         promote one of the replicas in that group. You can
         effectively prevent specific replicas from being promoted to
         primary in case of a failover by assigning a low-priority
         tier (high number) to these replicas. This is useful in cases
         where specific replicas are receiving heavy use by an
         application and failing over to one of them would negatively
         impact a critical application.

240. Backup retention period: 1 - 35 days (i.e. always on)

241. Encryption: optional, applies to cluster's storage volume, data,
     indexes, logs, automated backups, and snapshots

242. Access control

     1. DocumentDB does not support resource-based policies.

     2. IAM Policies control access to DocumentDB API actions and
         DocumentDB clusters and instances, not databases.

     3. Use **db.grantRolesToUser** command is used for assigning
         [DocumentDB database
         roles](https://docs.aws.amazon.com/documentdb/latest/developerguide/role_based_access_control.html)
         to DocumentDB database users.

     4. E.g. **db.grantRolesToUser("Bob", ({role: "read", db:
         "production"}))**.

243. Security DocumentDB

     1. Set the "**tls**" parameter to "**disabled**" turns off
         transport layer security (TLS) for client connections. This
         would not speed up query results, but would decrease the
         security of the database.

244. Upgrade: Auto OS/security patching, manual Engine Version Upgrade

245. Performance tuning for DocumentDB

     1. **DocumentDB Profiler** feature can be enabled to log the
         details (including execution time) of MongoDB operations to
         CloudWatch Logs.

     2. **CloudWatch Logs Insights** can then be used to analyze the
         data and investigate slow queries.
         ([Ref](https://docs.aws.amazon.com/documentdb/latest/developerguide/profiling.html))

     3. When a **DocumentDB cluster** takes a long time to return query
         results, use the **db.runCommand** with MongoDB **explain()**
         method to obtain a detailed **query execution plan** and
         insight into the **query performance**.

     4. When a **DocumentDB cluster** takes a long time to return query
         results:

         1. Create an index on the field being queried (speeds up the
             query operations).

         2. Set the "**secondaryPreferred**" read preference option
             distributes requests to read replicas. This is
             recommended as it increases performance efficiency of the
             database cluster.

             1.  Set the "**primaryPreferred**" read preference option
                 routes read operations to the primary instance. This
                 would not increase efficiency of query. It may
                 decrease performance of the cluster since all
                 requests would be going to the primary instance.

     5. How to identify blocked queries on a DocumentDB cluster?

         1. MongoDB **currentOp** command can be used to list queries
             that are either blocked or executing longer than a
             specified time
             ([Ref](https://docs.aws.amazon.com/documentdb/latest/developerguide/user_diagnostics.html#user_diagnostics-query_terminating)).

---
## Neptune

246. Neptune is a fully-managed, purpose-built, high-performance graph
     database optimized for storing billions of relationships (highly
     connected datasets) and querying the graph with milliseconds
     latency.

247. Neptune leverages operational technology that is shared with RDS
     (Aurora).

248. Max **64 TiB** Neptune cluster volume, max **64 TiB** graph size.
     Interface: **subset of Gremlin and SPARQL**

249. Database port: **8182**

250. Neptune supports query languages:

     1. **Apache TinkerPop Gremlin**, is a graph traversal language
         used with the **Property Graph (PG) model.**

     2. **RDF/SPARQL**, is the query language used with **RDF** models
         to describe relationships between graph objects.

251. A cluster consists of **0 - 16 instances**.

     1. One **primary DB instance**, supports read/write
         operations**,** performs all the data modifications to the
         **cluster volume**.

     2. A **cluster volume** is where data is stored. It is a single,
         virtual volume that uses solid-state disk (SSD) drives and
         designed for reliability and high availability. The cluster
         volume consists of copies of the data across 3 AZs in a
         single Region.

     3. **0-15 Read replicas**, support only **read**
         operations,connect to the same storage volume as the primary
         instance.

252. **Endpoints of a cluster**
     ([Ref](https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-endpoints.html))

     1. One **cluster endpoint**, connects to the current primary DB
         instance for the DB cluster.

         1. Example endpoint:
             sample-cluster.cluster-123456789012.us-east-1.neptune.amazonaws.com:8182

         2. You use the cluster endpoint for all write operations on
             the DB cluster, including inserts, updates, deletes, and
             DDL (data definition language) and DML (data manipulation
             language) changes.

         3. You can also use the cluster endpoint for read operations,
              such as queries.

     2. One **reader endpoint,** directs each connection request to one
         of the Neptune replicas.

         1. Example endpoint:
             sample-cluster.cluster-ro-123456789012.us-east-1.neptune.amazonaws.com:8182

         2. If the cluster only contains a primary instance and no read
             replicas, the reader endpoint connects to the primary
             instance. In that case, you can also do write-operations
             through the endpoint.

         3. The reader endpoint provides **round-robin routing** for
              read-only connections to the DB cluster. Use the reader
              endpoint for read operations, such as queries.

              1.  The reader endpoint round-robin routing works by
                  changing the host that the DNS entry points to. Each
                  time you resolve the DNS, you get a different IP,
                  and connections are opened against those IPs. After
                  a connection is established, all the requests for
                  that connection are sent to the same host.

              2.  The client must create a new connection and resolve
                  the DNS record again to get a connection to a
                  potentially different read replica.

              3.  **Problem**: DNS caching for clients or proxies
                  resolves the DNS name to the same endpoint from the
                  cache. This is a problem for both round robin
                  routing and failover scenarios.

                  1. **Disable any DNS caching settings to force DNS
                      resolution each time.**

         4. Neptune does not **load balance**. The reader endpoint only
             directs connections to available Neptune replicas in a
             Neptune DB cluster. It does not direct specific queries.

             1.  If you want to load balance queries to distribute the
                 read workload for a DB cluster, you must manage that
                 in your application. You must use **instance
                 endpoints** to connect directly to Neptune replicas
                 to balance the load.

     3. An **instance endpoint** per each instance, connects to a
         specific DB instance.

         1. Example endpoint:
             sample-instance.123456789012.us-east-1.neptune.amazonaws.com:8182

         2. The instance endpoint provides direct control over
             connections to the DB cluster, for scenarios where using
             the cluster endpoint or reader endpoint might not be
             appropriate.

253. Multi-AZ and automatic failover
     ([optional](https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-availability.html)),
     ([FAQs](https://aws.amazon.com/neptune/faqs/))

     1. Neptune stores copies of the data in a DB cluster across
         multiple AZs in a single Region, regardless of whether the
         instances in the DB cluster span multiple AZs.

     2. When you create Neptune replicas across AZs, Neptune
         automatically provisions and maintains them
         **synchronously**.

     3. Neptune automatically fails over to a Neptune replica in case
         the primary DB instance becomes unavailable.

     4. To maximize availability, place at least one replica in a
         different AZ from the primary instance.

     5. You can specify the fail-over priority for Neptune replicas.

     6. If you have a Neptune Replica, in the same or a different AZ,
         when failing over, Neptune flips the CNAME for your DB
         primary endpoint to a healthy replica, which is in turn is
         promoted to become the new primary. Start-to-finish, failover
         typically completes within **30 seconds**. Additionally, the
         read replicas endpoint does not require any CNAME updates
         during failover.

     7. If you do not have a Neptune Replica (i.e. single instance),
         Neptune will first attempt to create a new DB Instance in the
         same AZ as the original instance. If unable to do so, Neptune
         will attempt to create a new DB Instance in a different AZ.
         From start to finish, failover typically completes in under
         **15 minutes**.

254. **Neptune Storage Auto-Repair** detects segment failures in the
     SSDs and repairs segments using the data from the other volumes
     in the cluster
     ([Ref](https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-storage.html)).

255. Scaling - Storage Scaling
     ([Ref](https://docs.aws.amazon.com/neptune/latest/userguide/manage-console-performance-scaling.html))

     1. Storage will automatically grow, up to **64 TB**, in 10GB
         increments with no impact to database performance. The size
         of your cluster volume is checked on an hourly basis.

     2. Scaling down is NOT supported. Storage costs are based on the
         storage "high water mark" (the maximum amount allocated to
         your Neptune DB cluster at any time during its existence)

         1. Avoid ETL practices that create large amounts of temporary
             information, or that load large amounts of new data prior
             to removing unneeded older data.

         2. You can determine what the "high water mark" is currently
             for your Neptune DB cluster by monitoring the
             **VolumeBytesUsed** CloudWatch metric.

         3. If a substantial amount of your allocated storage is not
              being used, the only way to reset the high water mark is
              to export all the data in your graph and then reload it
              into a new DB cluster.

         4. Creating and restoring a snapshot does not reduce the
             amount allocated storage, because the physical layout of
             the underlying storage remains unchanged.

256. Scaling - Instance Scaling

     1. Scale your Neptune DB cluster as needed by modifying the DB
         instance class for each DB instance in the DB cluster.

257. Scaling - Read Scaling

     1. Create up to **15 Neptune replicas** in the DB cluster. Neptune
         replicas don't have to be of the same DB instance class as
         the primary instance.

258. Loading data into Neptune
     ([Ref](https://docs.aws.amazon.com/neptune/latest/userguide/get-started-loading.html))

     1. From S3

             # import.sh
             curl -X POST
             -H 'Content0Type: application/json'
             my-neptune-database.xxxxxx.us-east-1.neptune.amazonaws.com:8182/loader
             -d '
             {
             "source": "s3://database-demp/neptune/",
             "format": "csv",
             "iamRoleArn":
             "arn:aws:iam::xxxxxxxxxxxx:role/NeptuneLoadFromS3",
             "Region": "us-east-1",
             "failOnError": "FALSE",
             "Parallelism": "MEDIUM"
             }'

259. Database Cloning (similar as RDS)

260. Neptune Streams

261. Backup and Restore

     1. Automatic Backups: Backup retention period: **1 - 35 days
         (always on)**

         1. Neptune automatically monitors and backs up your database
             to S3, enabling granular PITR.

         2. If you delete a DB cluster, all its automated backups are
             deleted at the same time and cannot be recovered.

         3. Monitor CloudWatch metric BackupRetentionPeriodStorageUsed

     2. Manual Snapshots

         1. Manual snapshots are not deleted when the cluster is
             deleted.

         2. Monitor CloudWatch metric SnapshotStorageUsed

     3. CloudWatch metric TotalBackupStorageBilled =
         BackupRetentionPeriodStorageUsed + SnapshotStorageUsed

262. Encryption: optional, applies to all logs, backups, and snapshots
     ([Ref](https://docs.aws.amazon.com/neptune/latest/userguide/encrypt.html))

     1. If a KMS key identifier is not provided, Neptune uses your
         **default RDS encryption key (aws/rds)** for your new Neptune
         DB instance.

     2. After you create an encrypted Neptune DB instance, you cannot
         change the encryption key for that instance.

         1. If Neptune loses access to the encryption key for a Neptune
             DB instance (e.g. when Neptune access to a key is
             revoked), the encrypted DB instance is placed into a
             **terminal** state and **can only be restored from a
             backup**. We strongly recommend that you always enable
             backups for encrypted NeptuneDB instances to guard
             against the loss of encrypted data in your databases.

     3. You cannot enable encryption for a DB instance after it has
         been created.

     4. You cannot create an encrypted Neptune replica for an
         unencrypted Neptune DB cluster. You cannot create an
         unencrypted Neptune replica for an encrypted Neptune DB
         cluster.

     5. Encrypted Read Replicas must be encrypted with the same key as
         the source DB instance.

     6. You cannot convert an unencrypted DB cluster to an encrypted
         one.

         1. However, you can restore an unencrypted DB cluster snapshot
             to an encrypted DB cluster. To do this, specify a KMS
             encryption key when you restore from the unencrypted DB
             cluster snapshot.

263. Secure access to and within a Neptune database
     ([Ref](https://docs.aws.amazon.com/security/))

     1. 256-bit AES data encryption

     2. AWS IAM

     3. HTTPS

264. IAM DB Authentication: optional
     ([Ref](https://docs.aws.amazon.com/neptune/latest/userguide/iam-auth.html))

     1. The **neptune-db:** prefix and the **neptune-db:*** action are
         only for IAM DB Authentication. They aren't valid in any
         other context.

265. Upgrade: manual Engine Version Upgrade: using latest cluster engine
     version / older cluster engine version

266. Performance tuning

     1. Neptune does not require you to create specific indices to
         achieve good query performance.

     2. With support for up to **15 read replicas**, Neptune can
         support 100,000s of queries per second.

     3. Neptune uses query optimization for both SPARQL queries and
         Gremlin traversals.

---
## CloudFormation, Secret Manager and SSM parameter store

267. **Change sets** enable the preview of proposed changes to a stack
     in order to assess the impact on existing resources.

268. **StackSets** facilitate deployment and management of
     CloudFormation templates across multiple AWS accounts.

     1. A manual DB snapshot can be shared privately with other AWS
         accounts. StackSets extends the functionality of stacks by
         enabling you to create, update, or delete stacks across
         multiple accounts and Regions with a single operation from an
         administrator account.

269. **Stack Policies** can be used to deny actions on specific stack or
     resources to protect them from unintended modifications.

     1. E.g. To prevent accidental replacements or deletions of the production
        DDB when a template update is applied

            {"Effect": "Deny", "Action": "Update:*", "Principal": "*",
            "Resource": "LogicalResourceId/ProductionDatabase" }

270. **CloudFormation Registry** provides a listing of available
     CloudFormation providers in an account that can be used in
     CloudFormation templates.

271. CloudFormation **Rolling updates** dictate how CloudFormation
     handles deployment updates to resources.

272. Secrets Manager can be used to securely store, retrieve, and
     automatically rotate database credentials.

273. Systems Manager Parameter Store does not provide automatic
     credentials rotation capability.

274. KMS is used for management of cryptographic keys.

275. Resource Access Manager service is used for managing access to AWS
     resources between multiple accounts.

276. Secrets Manager is used to store the database credentials for a RDS
     PostgreSQL. How soon after enabling automatic rotation will the
     credential first be rotated?

     1. **Immediately** - The first rotation will happen immediately,
         so you need to make sure any applications which rely on this
         secret have been updated to retrieve it from Secrets Manager
         otherwise they will **no longer be able to access** the
         database.

277. Secrets Manager offers the ability to automatically rotate your
     secrets and passwords, keeping in line with normal 30- and 60-day
     rotation guidelines that many corporations will have.
     ([Ref](https://aws.amazon.com/blogs/security/rotate-amazon-rds-database-credentials-automatically-with-aws-secrets-manager/))

     1. This functionality has been integrated with RDS, Redshift, and
         DocumentDB.

     2. And the most powerful feature of all is that all these
         interactions can be implemented as simple **API calls**.

278. Dynamic reference patterns

     1. SSM parameter (e.g. use version 2 of the S3AccessControl
         parameter)

            "{{resolve:ssm:S3AccessControl:2}}"

     2. SSM secure string parameter

            "{{resolve:ssm-secure:IAMUserPassword:10}}"

     3. Secrets Manager secret

            '{{resolve:secretsmanager:MySecret:SecretString:password:v123}}'

     4. Secrets Manager secret (of another account)

           '{{resolve:secretsmanager:arn:aws:secretsmanager:us-west-2:123456789012:secret:MySecretName-asd123:SecretString:password:AWSPENDING}}'

280. Which option would you use in CloudFormation to reference version 2
     of a parameter named S3AccessControl that is stored in plain
     text?

        "AccessControl": "{{resolve:ssm:S3AccessControl:2}}"

281. Secrets Manager can be used for managing DB credentials when
     deploying an RDS instance with CF template.

         {
           "MyRDSInstance": {
              "Type": "AWS::RDS::DBInstance",
              "Properties": {
                "DBName": "MyRDSInstance",
                "AllocatedStorage": "20",
                "DBInstanceClass": "db.t2.micro",
                "Engine": "mysql",
                "MasterUsername": "{{resolve:secretsmanager:MyRDSSecret:SecretString:username}}",
                "MasterUserPassword": "{{resolve:secretsmanager:MyRDSSecret:SecretString:password}}"
             }
           }
         }

282. When using CodeBuild to deploy an application that requires to
     connect to an external database with database credentials being
     managed by Secrets Manager, the reference to the database
     credentials can be specified in the [**BuildSpec file**](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html), as a specification of build commands and settings.

            env:
              secrets-manager:
                key: secret-id:json-key:version-stage:version-id

---
## IAM Database Authentication, Encryption

283. IAM database authentication (
     [RDS Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html),
     [Aurora Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/UsingWithRDS.IAMDBAuth.html),
     [Neptune Ref](https://docs.aws.amazon.com/neptune/latest/userguide/iam-auth-enable.html) )

     1. Support only for **RDS MySQL, RDS PostgreSQL, Aurora MySQL,
         Aurora PostgreSQL, Neptune**

     2. Authenticate to your DB cluster using **IAM Database
         Authentication** (**token** instead of a password).

     3. Each token expires **15 minutes** after creation.

     4. Use IAM database authentication when your application requires
         fewer than 200 new IAM database authentication connections
         per second.

     5. IAM database authentication is NOT supported for CNAMEs.

     6. Aurora **MySQL parallel query** does NOT support IAM database
         authentication.

284. To enable IAM database authentication

     1. Enable **IAM database authentication** on the Aurora cluster.

     2. To allow an IAM user or role to connect to your DB
         cluster, you must create an IAM policy and attach the policy
         to an IAM user or role.

            "Action": "rds-db:connect",
            "Resource": "arn:aws:rds-db:us-east-2:1234567890:dbuser:cluster-ABCDEFGHIJKL01234/db_userx"

            # Or connecting to a database through RDS Proxy

            "Resource": "arn:aws:rds-db:us-east-2:1234567890:dbuser:prx-ABCDEFGHIJKL01234/db_userx")

            # db_userx is the name of the database account to associate with IAM authentication.

     3. Create a **database account** using IAM authentication

        1. The specified database account should have the same name as the
            IAM user or role.

        2. No need to assign database passwords to the user accounts you
            create.

        3. If you remove an IAM user that is mapped to a database account,
             you should also remove the database account with the DROP
             USER statement.

        4. **MySQL**: authentication is handled by
            **AWSAuthenticationPlugin** - Connect to the DB cluster, and:

                CREATE USER db_userx IDENTIFIED WITH AWSAuthenticationPlugin AS 'RDS';

        5.  **PostgreSQL**: connect to the DB cluster, create database users and
            then grant them the **rds_iam** role.

                CREATE USER db_userx;
                GRANT rds_iam TO db_userx;

285. To authenticate and access a RDS MySQL database instance
     ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.Connecting.AWSCLI.html))

     1. Generate an authentication token using **aws rds
         generate-db-auth-token** CLI command.

            RDSHOST="rdsmysql.cdgmuqiadpid.us-west-2.rds.amazonaws.com"

            TOKEN="$(**aws rds generate-db-auth-token** --hostname $RDSHOST
                --port 3306 --region us-west-2 --username jane_doe)"

            mysql --host=$RDSHOST --port=3306
                --ssl-ca=/sample_dir/rds-combined-ca-bundle.pem
                --enable-cleartext-plugin --user=jane_doe --password=$TOKEN


286. A company security policy mandates that all connections to
     databases must be encrypted. How should the user configure the
     connection parameters so that the client connection is protected
     against man-in-the-middle attack?

     1. **Aurora MySQL** DB cluster (using **mysql** to connect):
         ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Security.html#AuroraPostgreSQL.Security.SSL))
        <br> `--ssl-mode=verify-full --ssl-ca=/home/myuser/rds-combined-ca-bundle.pem`

     2. **Aurora PostgreSQL** DB cluster: Set `rds.force_ssl` parameter to **1**

     3. **RDS SQL Server**: `encrypt=true;trustServerCertificate=false`

     4. **RDS Oracle**: Set `ssl_server_dn_match` property to `true`
         ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Appendix.Oracle.Options.SSL.html))

287. The root certificate for enforcing all connections to RDS databases
     to be encrypted in transit using SSL/TLS
     ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html))

     1. Download it from
         [https://s3.amazonaws.com/rds-downloads/rds-ca-2019-root.pem](https://s3.amazonaws.com/rds-downloads/rds-ca-2019-root.pem).

---
## Network Troubleshooting

288. Three types of SGs are used with RDS
     ([Ref](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html))

     1. A **VPC security group** controls access to DB instances and
         EC2 instances inside a VPC.

     2. A **DB security group** controls access to EC2-Classic DB
         instances that are not in a VPC.

     3. An **EC2-Classic security group** controls access to an EC2
         instance.

289. If your DB instance is in a VPC but isn't publicly accessible, you
     can also use an AWS Site-to-Site VPN connection or an Direct
     Connect connection to access it from a private network.

290. What solution would establish a connection to the Aurora Serverless
     cluster from the Lambda function?

     1. Aurora Serverless DB cluster can't have a public IP address,
         and can only be accessed from within a VPC. Therefore we
         require to connect the Lambda to the private VPC using an
         ENI.
         ([Ref](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html))

291. An application consists of a front end hosted on EC2 instances in a
     public VPC subnet and an RDS database in a private VPC subnet.
     When attempting to establish a connection to the database, the
     application times out.

     1. VPC security group on the RDS database subnet must be
         configured to allow inbound traffic from the public subnet on
         the database port.

292. Which would be the recommended subnet for hosting an RDS database
     instance in this VPC?

            Subnet1 contains a route table entry with destination: 0.0.0.0/0 and
            target: VPC Internet Gateway ID

            Subnet2 contains a route table entry with destination 0.0.0.0/0 and
            target: NAT Gateway ID

            Subnet3 contains an EC2 instance that serves as a bastion host

            1. Subnet 2. Security best practice would state that RDS database
                instances should be deployed to a private subnet. A private subnet
                would only have private IP's with no direct access to the public
                internet. Outbound connectivity would be provided via a NAT
                gateway.

293. A company business intelligence team has a number of reporting
     applications deployed on EC2 instances in their AWS account. The
     company data warehouse team has provisioned a new set of
     databases using RDS in a different AWS account. What is the
     optimal solution to achieve secure and reliable connectivity from
     the business intelligence applications to the new RDS databases?

     1. VPC Peering is used to establish connectivity between two VPCs
         over Amazon's backbone network.

         1. PrivateLink endpoints are used to integrate AWS services to
             VPC without the use of IGW.

         2. Site-to-site VPN is not the optimal solution as it requires
             creation of VPN gateways.

         3. DX can be used to establish secure private connections
              between on-premise network and VPC over a dedicated
              line.

294. A company is migrating their on-premise data warehouse to Amazon
     Redshift. What methods can be used to establish a private
     connection from on-premise network to Amazon Redshift?

     1. Direct Connect can be used to establish a secure and private
         connection between on-premise network and VPC over a
         dedicated line.

     2. Site-to-site VPN can be used to establish a secure and private
         connection between an on-premise network and VPC over the
         Internet.

295. A company is migrating their on-premise MongoDB database to Amazon
     DynamoDB. Security team mandates that all data must be
     transferred over a dedicated, private, and secured connection
     with no data transport occurring over the public Internet. What
     AWS services must be part of the solution?

     1. Direct Connect can be used to establish a secure and private
         connection between on-premise network and VPC over a
         dedicated line.

     2. PrivateLink Gateway Endpoint is used to integrate DynamoDB to
         VPC without the use of IGW.

296. A solution architect would like to integrate a RDS for SQL Server
     instance with an existing Active Directory Domain.

     1. **AWS Directory Service** provides Microsoft Active Directory
         (AD) directory service that can be used to create an Active
         Directory environment in the AWS cloud. Other applications
         can join the provided domain and access the RDS for SQL
         Server instances in that same domain.
         ([Ref-1](https://aws.amazon.com/blogs/database/integrate-amazon-rds-for-sql-server-db-instances-with-an-existing-active-directory-domain/))
         ([Ref-2](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_SQLServerWinAuth.html))

---
## Choosing the right database technology

298. **ACID** (Atomicity, Consistency, Isolation, Durability)

     1. Require strong consistency
     2. **Aurora**, **DynamoDB transactions feature**, **Neptune**

299. **BASE** (Basically, Available, Soft state, Eventually consistent)

     1. Require performance, extremely low latency
     2. **DynamoDB**, **ElastiCache**

300. Frequent access to data over an extended period

     1. **DynamoDB** offers millisecond response rates for customer
         profiles and other active, fast-growing key-value datasets.

301. Short-term, low-latency access to temporary data

     1. **ElastiCache** is an in-memory data store for sub-millisecond
         access to temporary data such as game user sessions.

302. Infrequent access to an older dataset

     1. **Redshift Spectrum** can query data stored in S3 and join it
         data stored locally in Redshift tables. This allows you to
         store data at the lower S3 prices when appropriate.

303. **ElastiCache** vs. **RDS Read Replica**

     1. ElastiCache provides sub-millisecond response for read queries.
     2. Read Replica performance is dependent on the instance size. ElastiCache offers a better read performance solution.

304. **Neptune** use cases

     1. Make product recommendations
     2. Social networking sites can suggest relevant connections
     3. Fraud detection
     4. Example 1: Real-time streaming
         1. Collect (Kinesis Data Streams) -> Process (Lambda) -> Store (Neptune)
     5. Example 2: RSS keyword capture
         1. Parse (Comprehend) -> Raw store (S3 data lake) -> Process (Lambda) -> Final Store (Neptune)

305. **Athena** vs. **Redshift Spectrum** vs. **Aurora**

     1. Athena can query data in S3 directly using SQL query syntax. It is serverless, requiring no infrastructure.
     2. Redshift Spectrum can be used for complex reporting. Redshift Spectrum requires Redshift cluster, thus requiring additional infrastructure costs.
     3. Aurora requires infrastructure to store the data.

---
## Keyspaces

320. Interface: **CQL (Cassandra Query Language)**

321. In Cassandra, a keyspace is a grouping of tables that are related and used by your applications to read and write data.

322. In each Keyspaces table in Cassandra, there will be a primary key that consists of a **partition key** and **one or more columns**.

323. How can you run queries using CQL?

     1. Programmatically using an Apache 2 licensed Cassandra client driver.
     2. Through the CQL editor in the Amazon Keyspaces dashboard within the AWS management console.
     3. On a CQLSH client.

---
## QLDB

324. With QLDB, you can rest assured that nothing has changed or can be changed through the use of a **database journal**, which is configured as **append-only**.

325. Data for a QLDB database is placed into tables of **Amazon Ion documents**.
