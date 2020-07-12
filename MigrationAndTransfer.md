# Migration & Transfer

Table of Contents
- [Database Migration Service (DMS)](#database-migration-service-dms)
- [Snowball and Snowmobile](#snowball-and-snowmobile)


## Database Migration Service (DMS)

- DMS allows to quickly and securely migrate databases to AWS and keep an operational source database during the
  migration.

- The source database remains fully operational during the migration, minimizing downtime to applications that rely on
  the database. 

- DMS can migrate data between most common commercial and open-source databases.

- The service supports homogenous migrations such as Oracle to Oracle, as well as heterogeneous migrations between
  different database platforms, such as Oracle to Amazon Aurora or Microsoft SQL Server to MySQL. 

- DMS allows you to stream data to Amazon Redshift from any of the supported sources including Amazon Aurora,
  PostgreSQL, MySQL, MariaDB, Oracle, SAP ASE, and SQL Server, enabling consolidation and easy analysis of data in the
  **petabyte-scale** data warehouse. 

- DMS can be used for continuous data replication with high availability.

- Either the source or the target database (or both) need to reside in RDS or on EC2.

- **Replication between on-premises to on-premises databases is not supported.**

- Migrations can be from on-premises databases to Amazon RDS or Amazon EC2, databases running on EC2 to RDS, or vice
  versa, as well as from one RDS database to another RDS database.


## Snowball and Snowmobile

**Snowball**

- Snowball is a petabyte-scale data transport solution that uses and AWS-provided secure transfer appliance (the “Snowball”).
- Snowball is a physical device shipped from AWS.
- Snowball can quickly move large amount of data (10 TB or more) into and out of the AWS cloud.

- Encryption
  - Snowball can encrypt your data at load. Encryption is enforced, protecting your data at rest and in physical transit.

- Two sizes: 
  - All regions have an 80 TB option.
  - US regions also have a 50 TB option.

- Pricing
  - First 10 days of onsite usage are free* and each extra onsite day is $15. 
  - Data ingress (transfer IN to S3) is free. 
  - Data egress (transfer OUT of S3) is priced by region.

**Snowmobile**

- Snowmobile offers data transfer requiring up to 100 Petabytes.
- Snowmobile is equivalent to 1250 Snowball devices.
- Snowmobile makes it easy to move massive volumes of data to the cloud, including video libraries, image repositories, or even a complete data center migration. 
- Transferring data with Snowmobile is secure, fast, and cost-effective.



