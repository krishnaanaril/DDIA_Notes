# Summary

In this chapter we explored different ways of partitioning a large dataset into smaller
subsets. Partitioning is necessary when you have so much data that storing and pro‐
cessing it on a single machine is no longer feasible.

The goal of partitioning is to spread the data and query load evenly across multiple
machines, avoiding hot spots (nodes with disproportionately high load). This
requires choosing a partitioning scheme that is appropriate to your data, and reba‐
lancing the partitions when nodes are added to or removed from the cluster.
We discussed two main approaches to partitioning:

• Key range partitioning, where keys are sorted, and a partition owns all the keys
from some minimum up to some maximum. Sorting has the advantage that effi‐
cient range queries are possible, but there is a risk of hot spots if the application
often accesses keys that are close together in the sorted order.
In this approach, partitions are typically rebalanced dynamically by splitting the
range into two subranges when a partition gets too big.

• Hash partitioning, where a hash function is applied to each key, and a partition
owns a range of hashes. This method destroys the ordering of keys, making range
queries inefficient, but may distribute load more evenly.

When partitioning by hash, it is common to create a fixed number of partitions
in advance, to assign several partitions to each node, and to move entire parti‐
tions from one node to another when nodes are added or removed. Dynamic
partitioning can also be used.

Hybrid approaches are also possible, for example with a compound key: using one
part of the key to identify the partition and another part for the sort order.

We also discussed the interaction between partitioning and secondary indexes. A sec‐
ondary index also needs to be partitioned, and there are two methods:

• Document-partitioned indexes (local indexes), where the secondary indexes are
stored in the same partition as the primary key and value. This means that only a
single partition needs to be updated on write, but a read of the secondary index
requires a scatter/gather across all partitions.

• Term-partitioned indexes (global indexes), where the secondary indexes are parti‐
tioned separately, using the indexed values. An entry in the secondary index may
include records from all partitions of the primary key. When a document is writ‐
ten, several partitions of the secondary index need to be updated; however, a read
can be served from a single partition.

Finally, we discussed techniques for routing queries to the appropriate partition,
which range from simple partition-aware load balancing to sophisticated parallel
query execution engines.

By design, every partition operates mostly independently—that’s what allows a parti‐
tioned database to scale to multiple machines. However, operations that need to write
to several partitions can be difficult to reason about: for example, what happens if the
write to one partition succeeds, but another fails? We will address that question in the
following chapters.