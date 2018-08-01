---
title: Word Count & SparkR REPL Examples
author: Robert Hryniewicz
tutorial-id: 392
experience: Beginner
persona: Developer
source: Hortonworks
use case: Data Discovery
technology: Apache Spark
release: hdp-2.6.5
environment: Sandbox
product: HDP
series: HDP > Develop with Hadoop > Apache Spark
---

# Word Count & SparkR REPL Examples

## Introduction

This tutorial will get you started with a couple of Spark REPL examples

-   How to run Spark word count examples
-   How to use SparkR

## Prerequisites

This tutorial assumes that you are running an HDP Sandbox.

Please ensure you complete the prerequisites before proceeding with this tutorial.

-   Downloaded and Installed the [Hortonworks Sandbox](https://hortonworks.com/products/sandbox/)
-   Reviewed [Learning the Ropes of the HDP Sandbox](https://hortonworks.com/tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)

## Outline

-   [WordCount Example](#wordcount-example)
-   [SparkR Example](#sparkr-example)

-   [Further Reading](#further-reading)


## A Dataset WordCount Example

**Copy input file for Spark WordCount Example**

Upload the input file you want to use in WordCount to HDFS. You can use any text file as input. In the following example, log4j.properties is used as an example:

If you haven't already, switch to user *spark*:

~~~ bash
su spark
~~~

Next,

~~~ bash
cd /usr/hdp/current/spark2-client/
hdfs dfs -copyFromLocal /etc/hadoop/conf/log4j.properties /tmp/data.txt
~~~

**Run the Spark shell:**

~~~ bash
./bin/spark-shell 
~~~

Output similar to the following will be displayed, followed by a `scala>` REPL prompt:

~~~ bash
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.3.1.3.0.0.0-1634
      /_/

Using Scala version 2.11.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_181)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
~~~

Read data and convert to Dataset

~~~ js
val data = spark.read.textFile("/tmp/data.txt").as[String]
~~~

Split and group by lowercased word

~~~ js
val words = data.flatMap(value => value.split("\\s+"))
val groupedWords = words.groupByKey(_.toLowerCase)
~~~

Count words

~~~ js
val counts = groupedWords.count()
~~~

Show results

~~~ js
counts.show()
~~~

You should see the following output:

~~~
+--------------------+--------+
|               value|count(1)|
+--------------------+--------+
|                some|       1|
|hadoop.security.l...|       1|
|log4j.rootlogger=...|       1|
|log4j.appender.nn...|       1|
|log4j.appender.tl...|       1|
|hadoop.security.l...|       1|
|            license,|       1|
|                 two|       1|
|             counter|       1|
|log4j.appender.dr...|       1|
|hdfs.audit.logger...|       1|
|yarn.ewma.maxuniq...|       1|
|log4j.appender.nm...|       1|
|              daemon|       1|
|log4j.category.se...|       1|
|log4j.appender.js...|       1|
|log4j.appender.dr...|       1|
|        blockmanager|       1|
|log4j.appender.js...|       1|
|                 set|       4|
+--------------------+--------+
only showing top 20 rows
~~~

## An RDD WordCount Example

At the `scala` REPL prompt enter:

~~~ js
val file = sc.textFile("/tmp/data.txt")
val counts = file.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
~~~

Save `counts` to a file:

~~~ js
counts.saveAsTextFile("/tmp/wordcount")
~~~

To view the output in the shell type:

~~~ js
counts.count()
~~~

You should see an output screen similar to:

~~~ js
res2: Long = 364
~~~

To print the full output of the WordCount job type:

~~~ js
counts.take(10).foreach(println)
~~~

You should see an output screen similar to:

~~~ js
(Application,1)                                                                                               
(log4j.logger.org.apache.hadoop.yarn.server.resourcemanager.RMAppManager$ApplicationSummary=${yarn.server.reso
urcemanager.appsummary.logger},1)                                                                             
(Unless,1)                                                                                                    
(this,4)                                                                                                      
(hadoop.mapreduce.jobsummary.log.file=hadoop-mapreduce.jobsummary.log,1)                                      
(log4j.appender.RFA.layout.ConversionPattern=%d{ISO8601},2)                                                   
(AppSummaryLogging,1)                                                                                         
(log4j.appender.RMAUDIT.layout=org.apache.log4j.PatternLayout,1)                                              
(log4j.appender.DRFAAUDIT.layout=org.apache.log4j.PatternLayout,1)                                            
(under,4)
~~~

**Viewing the WordCount output with HDFS**

To read the output of WordCount using HDFS command:
Exit the Scala shell:

~~~ bash
:quit
~~~

View WordCount Results:

~~~ bash
hdfs dfs -ls /tmp/wordcount
~~~

You should see an output similar to:

~~~ bash
/tmp/wordcount/_SUCCESS
/tmp/wordcount/part-00000
/tmp/wordcount/part-00001
~~~

Use the HDFS `cat` command to see the WordCount output. For example,

~~~ bash
hdfs dfs -cat /tmp/wordcount/part-00000
~~~

## SparkR Example

**Install R**
Before proceeding, make sure that R is installed.
As `root`:

~~~ bash
cd /usr/hdp/current/spark2-client
sudo yum update
sudo yum install R
~~~

If you haven't done so already in previous sections, make sure to move *people.json* to HDFS:

~~~ bash
hdfs dfs -copyFromLocal examples/src/main/resources/people.json /tmp/people.json
~~~

**Launch SparkR**

Switch to *spark* user:

~~~ bash
su spark
~~~

Start SparkR:

~~~ bash
./bin/sparkR
~~~

First, create a DataFrame using sample dataset `faithful`:

~~~ js
df1 <- createDataFrame(faithful)
~~~

List the first few lines:

~~~ js
head(df1)
~~~

You should see an output similar to the following:

~~~ js
...
eruptions waiting
1     3.600      79
2     1.800      54
3     3.333      74
4     2.283      62
5     4.533      85
6     2.883      55
>
~~~

Now, create another DataFrame from `people.json` dataset:

~~~ js
df2 <- read.df("/tmp/people.json", "json")
~~~

List the first few lines:

~~~ js
head(df2)
~~~

You should see an output similar to the following:

~~~ js
  age    name
1  NA Michael
2  30    Andy
3  19  Justin
~~~

For more SparkR examples see https://spark.apache.org/docs/latest/sparkr.html

To exit type:

~~~ js
q()
~~~

## Next steps

Next learn more about [Spark DataFrames and Datasets](https://hortonworks.com/tutorial/dataframe-and-dataset-examples-in-spark-repl/) that allow for higher-level data manipulation and abstract away optimization details.
