---
layout: default
title: HTAP Initiative
tags: [HTAP, database]
nav_order: {{ page.date }}
---


# What is the plan?

Now there are main classes of databases. One class is optimized for
OLTP, like Oracle, SQL Server, MySQL, etc. The other class is for OLAP
tasks. This class includes warehouse systems, like SAP HANA, Hive, Iceberg,
Hudi, etc. Usually we need these two classes of systems in business
operations. So the data flow between these two classes systems
requires extra work to keep the data consistent and ready. So our aim
is to design a HTAP system which takes OLTP and OLAP tasks into
consideration. This is not an easy task, but with good examples of
open source systems. We can address this task by adding the part
needed by OLTP to open source OLAP systems.


# The method that we will use.

OLTP systems can deal with random queries and data mutations very
quickly, because OLTP systems have delicate designed index and data
layout formats. These systems can pin-point a small set of data
quickly. Such design is something that we find missing in open source
OLAP systems. To make this task more practical we choose to enhance
Apache Iceberg and make necessary optimizations. Apache
Iceberg is one of the best datalake systems. It can be used along with Spark to
serve relational tables. Users can use SQL to run queries and modifications. The data in Apache
Spark is distributed into partitions, which is called partitioning. Partitioning works by assigning data with same values in partition
columns in the same partition. Besides partitoning Apache Iceberg utilizes json files and
manifest files (Avro format) to build an index. It is used to
boost data file location. This index structure is suitable for batch process
tasks which often go through whole data files. Considering OLTP tasks there are two limits
with such design. The first limit is that partition selection requires
experience. Inapproriate partition selection may severely deteriorate the
performance. The second limit is that there is no data level index. It is
not efficient for record operations because the index is for files.

The first optimization is to eliminate the need to select partition
columns. In order to skip partition selection we plan to organize the
data in levels which is similar to LSM-Tree. Data is first write to a
long struture accompanied by a in-memory index struture. After enough data is
accumulated, it is flushed to the first level. When the data in the
first level is accumulated data is merged into the second level. This
process is continued through all the levels of the data
organization. The next level also adds one dimension to partition the
data. For example the first level data is assigned to ranges, the
second level data is assigned to squares, the third level data is
assigned to cubes, and so on. These partition dimensions are selected
based on data distribution characteristics of the last level. To help
with that task we need to collect data statistics for data
columns. With such an orgnaization we can handle data mutation more
easily and also make the data distributed evenly.

The second optimization is to optimize the index. In order to quickly
pin-point any data we plan to build a two level index. The first level
is a index for data files. The second level is a per data file index,
which is used to locate a data record. When one wants to find a data
record, he/she can first check the data file index, then the per data
index to pin-point the data record.

With these two optimization the new system still can handle OLAP
tasks by going through data files as a whole. But with re-organization
of the data format and new indexes it can also handle record level
data change more efficiently. With the new indexes the new system also
can serve data queries efficiently.


# Other considerations

Beside these two optimizations we plan to use open source
components as many as possible. Parquet, Avro are good
candidates. These projects should be used as data file and log file
format. And a efficient and powerful language also can impact
efficiency a lot. Though Iceberg is mainly a Java project, Java is
tedious and requires a lot of typing. In contrast scala is more
compact and employ many modern language features. Scala program is
more compact and natural. Scala is also built into java
bytecode. It can interoperate on java code and libraries. So we can
use Scala to add optimization code. We recommend Scala as the development
language.
Since we will use the open source projects as the building blocks of
this initiative. I think it is better to make this initiative a open
source project, too. Anyone who is interested in our idea can
contribute to this project.

