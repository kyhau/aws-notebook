# Essential Big Data Tools on EMR

- Apache Hadoop on EMR
   - Hadoop is an open-source framework that processes massive datasets via distributed computing.
   - It relies on MapReduce to distribute the data processing across instances.
   - It implements HDFS to store data across multiple instances.

- Apache Spark
   - Spark is an open-source, distributed processing framework with an optimized DAG execution engine and caching of data in-memory.
   - Micro-batching, but can guarantee only-once deliver if configured.
   - Apache Livy, a tool that allows you to use a REST interface to interact with an EMR cluster running Spark.

- Apache HBase on HDFS
   - HBase is open source, non-relational, distributed database that runs on top of HDFS.
   - HBase is a key/value store. 
   - HBase is a Sparse, Consistent, Distributed, Multidimensional, Sorted map.

- Apache Hive on EMR
   - Hive is an open-source, data warehouse, and analytic package that runs on top of a Hadoop cluster and abstracts programming models away with scripts written in HiveQL.
   - Hive uses “Schema-on-read” (no need to predefined).
   - Hive on EMR supports direct integration with DynamoDB and S3. 
   - With Amazon EMR you can load table partitions automatically from S3, you can write data to tables in S3 without using temporary files, and you can access resources in S3, such as scripts for custom map/reduce operations and additional libraries.

- Apache Pig on EMR
   - Pig is an open-source library running on top of Hadoop, providing a scripting language (Pig Latin) that you can use to transform large data sets without having to write complex code in a lower level computer language like Java. 

- Apache HCatalog on EMR
   - HCatalog is table storage manager for Hadoop, allows you to access Hive metastore tables within Pig, Spark SQL, and/or custom MapReduce applications.

- Apache Oozie 
   - Oozie is a Workflow Scheduler for Hadoop; manages Hadoop jobs including Spark. 

Streaming

- Apache Flink
   - Flink is a streaming dataflow engine that can run real-time stream processing on high-throughput data sources. 
   - Flink supports event time semantics for out-of-order events, exactly-once semantics, backpressure control, and APIs optimized for writing both streaming and batch (processing) applications.
   - Can replay streams

- Apache Spark Streaming
   - Spark Streaming is an extension of the Spark API that is adapted to process streaming data.
   - Only-once delivery
   - Out-of-order data

- Apache Kafka
   - Ordered data
   - Batching processing

Configuration and Monitoring

- Ganglia Monitoring System on EMR
   - Ganglia is a scalable, distributed system designed to monitor and report on cluster/node performance. 

- Apache Zookeeper
   - Zookeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.

Other tools

- Apache Flume
   - Flume is a distributed, reliable, and available service for efficiently collecting, aggregating, and moving large amounts of log data. 
   - It has a simple and flexible architecture based on streaming data flows.
   - Use Flume to migrate the data into HDFS and Hive/HiveQL on top of Hadoop to query the data.

- Apache Sqoop
   - Sqoop is a tool for transferring data between S3, Hadoop, HDFS, and RDBMS databases.

- Apache Tez
   - A framework for creating DAGs. 
   - In some cases you might use it as an alternative to Hadoop MapReduce (Pig and Hive can use Tez as an alternative execution engine).

GUIs and Data Exploration

- Presto
   - Presto is a fast distributed SQL query engine designed for interactive analytic queries over large datasets from multiple sources.
   - HDFS/Hive, RDBMS (MySQL, PostgreSQL), NoSQL, Kakfa, etc.

- Apache Phoenix
   - Phoenix is used for OLTP and operational analytics for Hadoop. 
   - Phoenix uses SQL queries and JDBC APIs to work with a HBase data store.

- Apache Spark SQL
   - Allow you to query structured data in Spark programs using SQL or a DataFrame API.

- Apache Zeppelin
   - Zeppelin is a web-based notebook that enables data-driven, interactive data analytics and collaborative documents with SQL, Scala and more.
   - Zeppelin creates interactive and collaborative notebooks for data exploration using Spark.
   - You can use Scala, Python, SQL (using Spark SQL), or HiveQL to manipulate data

- Hue (Hadoop User Experience)
   - A web-based GUI that acts as a front-end for applications that run on your cluster. 
   - Allows you to use the Hue GUI to interact with tools like Hive and Pig.

- Jupyter, JupyterHub

Machine Learning Tools

- Apache Mahout
   - Mahout is a machine learning framework for Apache Hadoop, that has built-in tools for common use cases like clustering, classification, and recommender systems. 
   - Mahout employs the Hadoop framework to distribute calculations across a cluster, and now includes additional work distribution methods, including Spark.

- Apache MXNet
   - A library to help you build neural networks and other deep learning applications.

- SparkMLlib

- Tensorflow
   - An open-source library for machine intelligence and deep learning applications.

