---
title: Spark SQL Thrift Server Example
author: Robert Hryniewicz
tutorial-id: 393
experience: Intermediate
persona: Developer
source: Hortonworks
use case: Data Discovery
technology: Apache Spark
release: hdp-2.6.1
environment: Sandbox
product: HDP
series: HDP > Develop with Hadoop > Apache Spark
---

# Spark SQL Thrift Server Example

## Introduction

This is a very short tutorial on how to use SparkSQL Thrift Server for JDBC/ODBC access

## Prerequisites

This tutorial assumes that you are running an HDP Sandbox.

Please ensure you complete the prerequisites before proceeding with this tutorial.

-   Downloaded and Installed the [Hortonworks Sandbox](https://hortonworks.com/products/sandbox/)
-   Reviewed [Learning the Ropes of the HDP Sandbox](https://hortonworks.com/tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)

## SparkSQL Thrift Server Example

SparkSQL’s thrift server provides JDBC access to SparkSQL.

**Create logs directory**

As `root`:

~~~ bash
cd /usr/hdp/current/spark2-client
mkdir logs
chown spark:hadoop logs
~~~

**Start Thrift Server**

~~~ bash
./sbin/start-thriftserver.sh --master yarn-client --executor-memory 512m --hiveconf hive.server2.thrift.port=10015
~~~

**Connect to the Thrift Server over Beeline**

Launch Beeline:

~~~ bash
./bin/beeline
~~~

**Connect to Thrift Server and issue SQL commands**

On the `beeline>` prompt type:

~~~ sql
!connect jdbc:hive2://localhost:10015
~~~

When prompted for username and password just press `enter`.

You should see an output similar to the following:

~~~
Beeline version 1.21.2.3.0.0.0-1634 by Apache Hive
beeline> !connect jdbc:hive2://localhost:10015
Connecting to jdbc:hive2://localhost:10015
Enter username for jdbc:hive2://localhost:10015:
Enter password for jdbc:hive2://localhost:10015:
...
Connected to: Spark SQL (version 2.3.1.3.0.0.0-1634)
Driver: Hive JDBC (version 1.21.2.3.0.0.0-1634)
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://localhost:10015>
~~~

Once connected, try `show tables`:

~~~ sql
show tables;
~~~

You should see an output similar to the following:

~~~ bash
+-----------+------------+--------------+--+
| database  | tableName  | isTemporary  |
+-----------+------------+--------------+--+
+-----------+------------+--------------+--+
No rows selected (0.278 seconds)                                                                                         
0: jdbc:hive2://localhost:10015>
~~~

Note that the table list is empty since there are no sample tables.

Type `!exit` to exit beeline.

**Stop Thrift Server**

~~~ bash
./sbin/stop-thriftserver.sh
~~~
