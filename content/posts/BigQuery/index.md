---
title: "BigQuery: Behind the Scenes"
date: 2023-08-11T09:07:51+02:00
draft: true
---

- Serverless
- Distributed nature
- Columnar storage
  - Compression
  - Subset of columns
- Clustering
  - Which columns
  - Benefits for aggregations
  - Benefits for compression
- Partitioning
  - Helps with appending data
  - Filtering
- Immutability
  - Helps with backups, recovery and versioning


## Distributed
The core of BigQuery's storage system is its distributed nature. Instead of storing the data on a single machine or in a single data center, the data is split up. This allows BigQuery to scale to petabyte-scale datasets.

## Columnar
BigQuery stores its data in columns, rather than by rows. Columnar storage allows for more efficient compression, as the data in a column is often similar. For example, a column containing the names of countries will contain a lot of duplicate values, which can be compressed efficiently. This is in contrast to row-based storage, where each row contains a lot of different values, which are often not similar.

So what is compression exactly? It is just a way of storing data in a more efficient way. One of the most common compression techniques in columnar storage is Run-Length Encoding (RLE). It works by storing the value, and the number of consecutive rows in which this value is found. This is why it is often beneficial to sort the data before storing it in BigQuery, as it will increase the number of consecutive rows with the same value. One way of having BigQuery sort the data for you is by using the "cluster by" clause when creating a table. Another common compression technique is Dictionary Encoding. It works by storing the unique values in a column in a dictionary, and then storing the index of each value in the dictionary. This is why it is often beneficial to use integer values instead of strings, as it will reduce the size of the dictionary.

Columnar storage also means that the speed of analytics queries is improved when only requiring a subset of the columns, which is often the case. It significantly reduces the amount of I/O. I/O refers to the input/output operations that are required to read data from disk. It is often the bottleneck in analytics queries, as it is much slower than CPU operations.


## Clustering
So we have talked about clustering improving query performance. But why is this the case? BigQuery uses block pruning to eliminate blocks of data that do not need to be scanned. Each block of data has some metadata associated with it that includes the minimum and maximum values for the clustered columns. When a query is executed, BigQuery can quickly determine if a block contains relevant data by comparing the query predicates to this metadata. This leads to less I/O operations. 

Aggregations are also faster, as there is less shuffling of data going on. For example, if you want to calculate the average of a column, BigQuery can quickly determine the number of rows in each block, and then calculate the average of each block. It can then calculate the average of the averages to get the final result. This is much faster than having to shuffle all the data around to calculate the average.

## Partitioning
- Partitioning literally means dividing the data in segments. 
- Optimized data writes. Appending to a partitioned table is faster, as BigQuery only needs to write to the latest partition. This is in contrast to non-partitioned tables, where BigQuery needs to write to the entire table. This is especially important for streaming inserts, as BigQuery needs to write to the table as fast as possible.


## Immutability
BigQuery's storage system is immutable. This means that once a table is created, it cannot be modified. This is in contrast to traditional databases, where you can update or delete rows. This is one of the reasons why BigQuery is so fast, as it doesn't have to worry about locking or concurrency issues. Furthermore, it makes handling backups, versioning and recovery much easier. 


**Notes:**
- Capacitor is the proprietary columnar storage format that BigQuery uses. It is based on the Parquet format, but with some modifications to make it more suitable for BigQuery.





