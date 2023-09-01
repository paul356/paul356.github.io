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
quickly. Such design is something that we seldom find in open source
OLAP systems. But the open source OLAP systems have open source data
format and components. These components are interchangeable. We can
use the most suitable project to do one specific task. So our method
is reuse as many open source components as possible to build this HTAP
database. Only write new components when there is no suitable project
available.
More specifically the method is to enhance Apache Iceberg and add
record level indexes and more smart data mutation algorithms.
Apache Iceberg is built on top of opens source projects like Apache Parquet, Avro, HDFS
client, etc which is one of the best. It can be used along with Spark to
serve relational tables. Users can use SQL to do query and modifications. Apache
Iceberg is more suitable for OLAP/batch processing, because it has a
limited index structure. The index structure consists of json files
and manifest files (Avro format). It is used to quickly locate data
files. This index structure is more suitable for batch processing tasks which often go through whole data files.

