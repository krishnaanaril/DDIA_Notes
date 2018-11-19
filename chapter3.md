# Summary

In this chapter we tried to get to the bottom of how databases handle storage and
retrieval. What happens when you store data in a database, and what does the data‐
base do when you query for the data again later?

On a high level, we saw that storage engines fall into two broad categories: those opti‐
mized for transaction processing (OLTP), and those optimized for analytics (OLAP).
There are big differences between the access patterns in those use cases:

• OLTP systems are typically user-facing, which means that they may see a huge
volume of requests. In order to handle the load, applications usually only touch a
small number of records in each query. The application requests records using
some kind of key, and the storage engine uses an index to find the data for the
requested key. Disk seek time is often the bottleneck here.

• Data warehouses and similar analytic systems are less well known, because they
are primarily used by business analysts, not by end users. They handle a much
lower volume of queries than OLTP systems, but each query is typically very
demanding, requiring many millions of records to be scanned in a short time.
Disk bandwidth (not seek time) is often the bottleneck here, and column-
oriented storage is an increasingly popular solution for this kind of workload.
On the OLTP side, we saw storage engines from two main schools of thought:

• The log-structured school, which only permits appending to files and deleting
obsolete files, but never updates a file that has been written. Bitcask, SSTables,
LSM-trees, LevelDB, Cassandra, HBase, Lucene, and others belong to this group.

• The update-in-place school, which treats the disk as a set of fixed-size pages that
can be overwritten. B-trees are the biggest example of this philosophy, being used
in all major relational databases and also many nonrelational ones.
Log-structured storage engines are a comparatively recent development. Their key
idea is that they systematically turn random-access writes into sequential writes on
disk, which enables higher write throughput due to the performance characteristics
of hard drives and SSDs.

Finishing off the OLTP side, we did a brief tour through some more complicated
indexing structures, and databases that are optimized for keeping all data in memory.
We then took a detour from the internals of storage engines to look at the high-level
architecture of a typical data warehouse. This background illustrated why analytic
workloads are so different from OLTP: when your queries require sequentially scan‐
ning across a large number of rows, indexes are much less relevant. Instead it
becomes important to encode data very compactly, to minimize the amount of data
that the query needs to read from disk. We discussed how column-oriented storage
helps achieve this goal.

As an application developer, if you’re armed with this knowledge about the internals
of storage engines, you are in a much better position to know which tool is best suited
for your particular application. If you need to adjust a database’s tuning parameters,
this understanding allows you to imagine what effect a higher or a lower value may
have.

Although this chapter couldn’t make you an expert in tuning any one particular stor‐
age engine, it has hopefully equipped you with enough vocabulary and ideas that you
can make sense of the documentation for the database of your choice.