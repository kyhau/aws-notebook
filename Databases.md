# Databases

- [Quick Summary](#quick-summary)

---
## Quick Summary

1. Aurora
   - Minor Engine Version Upgrade: If enabled, done within the weekly maintenance window.
   - Major Engine Version Upgrade: Manual upgrade required
   - Backup retention period: 1-35 days, when you create or modify a DB cluster.
   - Encryption: If enabled, applies to underlying storage for DB clusters, automated backups, Read Replicas, and snapshots.
   - Multi-AZ: [optional](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)
2. RDS
   - OS/Security Patching: Done within weekly maintenance window. Such patching occurs infrequently (typically once every few months)
   - Minor Engine Version Upgrade: If enabled, done within the weekly maintenance window.
   - Major Engine Version Upgrade: Manual upgrade required
   - Backup retention period: 0 - 35 days. 0 = disables automated backups.
   - Encryption: If enabled, applies to underlying storage for DB clusters, automated backups, Read Replicas, and snapshots.
   - Multi-AZ: optional
3. DocumentDB 
   - OS/Security Patching: Done within weekly maintenance window. Such patching occurs infrequently (typically once every few months)
   - Engine Version Upgrade: Manually trigger the upgrade, options: upgrade now or upgrade at next window
   - Backup retention period: 1 - 35 days
   - Encryption: If enabled, applies to cluster’s storage volume, data, indexes, logs, automated backups, and snapshots
   - [Multi-AZ](https://docs.aws.amazon.com/documentdb/latest/developerguide/replication.html): When you create a DocumentDB cluster, depending upon the number of AZs in the subnet group (there must be at least two), DocumentDB provisions instances across the AZs.
4. DynamoDB
   - DynamoDB does not have the deletion protection feature.
   - Retention period: max 35 days (can't be modified) for PITR.
   - Multi-AZ, Multi-Region, Unlimited data volume, Interface: AWS API
   - Max item size: 400 KB
   - DynamoDB global tables automatically replicate data across 2 or more Regions, with full support for multi-master writes.
5. ElastiCache (Memcached)
   - OS/Security Patching: Done within weekly maintenance window. Such patching occurs infrequently (typically once every few months)
   - Engine Version Upgrade: Manually trigger the upgrade, options: upgrade now or upgrade at next window
   - Backup: [not supported](https://aws.amazon.com/elasticache/faqs/)
   - Encryption: [not supported](https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/SelectEngine.html)
   - Multi-AZ: [not supported](https://aws.amazon.com/elasticache/faqs/)
6. ElastiCache (Redis)
   - OS/Security Patching: Done within weekly maintenance window. Such patching occurs infrequently (typically once every few months)
   - Engine Version Upgrade: Manually trigger the upgrade, options: upgrade now or upgrade at next window
   - Backup retention period: 0 - 35 days. 
   - Encryption: If enabled, applies to disk during sync, backup and swap operations, backups stored in S3
   - Multi-AZ: optional https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/AutoFailover.html
7. Neptune 
   - Engine Version Upgrade: Using latest cluster engine version / Older cluster engine version , Manually trigger the upgrade, options: upgrade now or upgrade at next window
   - Backup retention period: 1 - 35 days, when you create or modify a DB cluster.
   - Encryption: If enabled, applies to all logs, backups, and snapshots
   - Multi-AZ: [optional](https://docs.aws.amazon.com/neptune/latest/userguide/feature-overview-availability.html)
8. Redshift  
   - Engine Version Upgrade: Using Current-or-Trailing-or-Preview cluster version / Older engine version , Cluster Redshift engine is periodically updated during its weekly maintenance window. Upgrade versions to choose: Current, Trailing, Preview
   - Backup retention period: 0 - 35 days. If you disable automated snapshots, Redshift stops taking snapshots and deletes any existing automated snapshots for the cluster.
   - Encryption: If enabled, applies to cluster’s storage volume, data, indexes, logs, automated backups, and snapshots.
   - Multi-AZ: Currently, Redshift only supports Single-AZ deployments. You can run data warehouse clusters in multiple AZ's by loading data into two Amazon Redshift data warehouse clusters in separate AZs from the same set of Amazon S3 input files. With Redshift Spectrum, you can spin up multiple clusters across AZs and access data in Amazon S3 without having to load it into your cluster. In addition, you can also restore a data warehouse cluster to a different AZ from your data warehouse cluster snapshots. https://aws.amazon.com/redshift/faqs/

