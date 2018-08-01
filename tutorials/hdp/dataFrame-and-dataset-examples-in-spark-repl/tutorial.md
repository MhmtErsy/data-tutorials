---
title: DataFrame and Dataset Examples in Spark REPL
author: Robert Hryniewicz
tutorial-id: 391
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

# DataFrame and Dataset Examples in Spark REPL

## Introduction

This tutorial will get you started with Apache Spark and will cover:

- How to use the Spark DataFrame & Dataset API
- How to use SparkSQL Thrift Server for JDBC/ODBC access

Interacting with Spark will be done via the terminal (i.e. command line).

## Prerequisites

This tutorial assumes that you are running an HDP Sandbox.

Please ensure you complete the prerequisites before proceeding with this tutorial.

-   Downloaded and Installed the [Hortonworks Sandbox](https://hortonworks.com/products/sandbox/)
-   Reviewed [Learning the Ropes of the HDP Sandbox](https://hortonworks.com/tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)

## Outline

-   [DataFrame API Example](#dataframe-api-example)
-   [DataSet API Example](#dataset-api-example)

-   [Further Reading](#further-reading)


## DataFrame API Example

 DataFrame API provides easier access to data since it looks conceptually like a Table and a lot of developers from Python/R/Pandas are familiar with it.

Assuming you start as `root` user, switch to user *spark*:

~~~ bash
su spark
~~~

Next, upload people.txt and people.json files to HDFS:

~~~ bash
cd /usr/hdp/current/spark2-client
hdfs dfs -copyFromLocal examples/src/main/resources/people.txt /tmp/people.txt
hdfs dfs -copyFromLocal examples/src/main/resources/people.json /tmp/people.json
~~~

Launch the Spark Shell:

~~~ bash
spark-shell
~~~

At a `scala>` REPL prompt, type the following:

~~~ js
val df = spark.read.json("/tmp/people.json")
~~~

Using `df.show`, display the contents of the DataFrame:

~~~ js
df.show
~~~

You should see an output similar to:

~~~ js
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~

Now, let's select "name" and "age" columns and increment the "age" column by 1:

~~~ js
df.select(df("name"), df("age") + 1).show()
~~~

This will produce an output similar to the following:

~~~ bash
+-------+---------+
|   name|(age + 1)|
+-------+---------+
|Michael|     null|
|   Andy|       31|
| Justin|       20|
+-------+---------+
~~~

To return people older than 21, use the filter() function:

~~~ js
df.filter(df("age") > 21).show()
~~~

This will produce an output similar to the following:

~~~ bash
+---+----+
|age|name|
+---+----+
| 30|Andy|
+---+----+
~~~

Next, to count the number of people of specific age, use groupBy() & count() functions:

~~~ js
df.groupBy("age").count().show()
~~~

This will produce an output similar to the following:

~~~ bash
+----+-----+
| age|count|
+----+-----+
|null|    1|
|  19|    1|
|  30|    1|
+----+-----+
~~~

**Programmatically Specifying Schema**

~~~ js
import org.apache.spark.sql._

// Create an RDD
val peopleRDD = spark.sparkContext.textFile("/tmp/people.txt")

// The schema is encoded in a string
val schemaString = "name age"

// Generate the schema based on the string of schema
val fields = schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, nullable = true))
val schema = StructType(fields)

// Convert records of the RDD (people) to Rows
val rowRDD = peopleRDD.map(_.split(",")).map(attributes => Row(attributes(0), attributes(1).trim))

// Apply the schema to the RDD
val peopleDF = spark.createDataFrame(rowRDD, schema)

// Creates a temporary view using the DataFrame
peopleDF.createOrReplaceTempView("people")

// SQL can be run over a temporary view created using DataFrames
val results = spark.sql("SELECT name FROM people")

// The results of SQL queries are DataFrames and support all the normal RDD operations
// The columns of a row in the result can be accessed by field index or by field name
results.map(attributes => "Name: " + attributes(0)).show()

~~~

This will produce an output similar to the following:

~~~ js
+-------------+  
|        value|                                              
+-------------+                                      
|Name: Michael|                                             
|   Name: Andy|                                                
| Name: Justin|
+-------------+
~~~

## DataSet API Example

The Spark Dataset API brings the best of RDD and DataFrames together, for type safety and user functions that run directly on existing JVM types.

Let's try the simplest example of creating a dataset by applying a *toDS()* function to a sequence of numbers.

At the `scala>` prompt, copy & paste the following:

~~~ js
val ds = Seq(1, 2, 3).toDS()
ds.show
~~~

You should see the following output:

~~~ bash
+-----+
|value|
+-----+
|    1|
|    2|
|    3|
+-----+
~~~

Moving on to a slightly more interesting example, let's prepare a *Person* class to hold our person data. We will use it in two ways by applying it directly on a hardcoded data and then on a data read from a json file.

To apply *Person* class to hardcoded data type:

~~~ bash
case class Person(name: String, age: Long)
val ds = Seq(Person("Andy", 32)).toDS()
~~~

When you type

~~~ bash
ds.show
~~~

you should see the following output of the *ds* Dataset

~~~ bash
+----+---+
|name|age|
+----+---+
|Andy| 32|
+----+---+
~~~

Finally, let's map data read from *people.json* to a *Person* class. The mapping will be done by name.

~~~ bash
val path = "/tmp/people.json"
val people = spark.read.json(path)
~~~

To view contents of people DataFrame type:

~~~ js
people.show
~~~

You should see an output similar to the following:

~~~ bash
+----+-------+
| age|   name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
~~~

Note that the *age* column contains a *null* value. Before we can convert our people DataFrame to a Dataset, let's filter out the *null* value first:

~~~ bash
val pplFiltered = people.filter("age is not null")
~~~

Now we can map to the *Person* class and convert our DataFrame to a Dataset.

~~~ bash
val pplDS = pplFiltered.as[Person]
~~~

View the contents of the Dataset type

~~~ bash
pplDS.show
~~~

You should see the following

~~~ bash
+------+---+
|  name|age|
+------+---+
|  Andy| 30|
|Justin| 19|
+------+---+
~~~

To exit type `:quit`

## Further Reading

Next, explore more advanced [SparkSQL commands in a Zeppelin Notebook]( https://hortonworks.com/tutorial/learning-spark-sql-with-zeppelin/).
