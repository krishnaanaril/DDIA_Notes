# Chapter 1: Reliable, Scalable, and Maintainable Applications

In this chapter, we have explored some fundamental ways of thinking about data-
intensive applications. These principles will guide us through the rest of the book,
where we dive into deep technical detail.

An application has to meet various requirements in order to be useful. There are
functional requirements (what it should do, such as allowing data to be stored,
retrieved, searched, and processed in various ways), and some nonfunctional require‐
ments (general properties like security, reliability, compliance, scalability, compatibil‐
ity, and maintainability). In this chapter we discussed reliability, scalability, and
maintainability in detail.

Reliability means making systems work correctly, even when faults occur. Faults can
be in hardware (typically random and uncorrelated), software (bugs are typically sys‐
tematic and hard to deal with), and humans (who inevitably make mistakes from
time to time). Fault-tolerance techniques can hide certain types of faults from the end
user.

Scalability means having strategies for keeping performance good, even when load
increases. In order to discuss scalability, we first need ways of describing load and
performance quantitatively. We briefly looked at Twitter’s home timelines as an
example of describing load, and response time percentiles as a way of measuring per‐
formance. In a scalable system, you can add processing capacity in order to remain
reliable under high load.

Maintainability has many facets, but in essence it’s about making life better for the
engineering and operations teams who need to work with the system. Good abstrac‐
tions can help reduce complexity and make the system easier to modify and adapt for
new use cases. Good operability means having good visibility into the system’s health,
and having effective ways of managing it.

There is unfortunately no easy fix for making applications reliable, scalable, or main‐
tainable. However, there are certain patterns and techniques that keep reappearing in
different kinds of applications. In the next few chapters we will take a look at some
examples of data systems and analyze how they work toward those goals.

# Chapter 2: Data Models and Query Languages

Data models are a huge subject, and in this chapter we have taken a quick look at a
broad variety of different models. We didn’t have space to go into all the details of
each model, but hopefully the overview has been enough to whet your appetite to
find out more about the model that best fits your application’s requirements.

Historically, data started out being represented as one big tree (the hierarchical
model), but that wasn’t good for representing many-to-many relationships, so the
relational model was invented to solve that problem. More recently, developers found
that some applications don’t fit well in the relational model either. New nonrelational
“NoSQL” datastores have diverged in two main directions:

1. Document databases target use cases where data comes in self-contained docu‐
ments and relationships between one document and another are rare.
2. Graph databases go in the opposite direction, targeting use cases where anything
is potentially related to everything.

All three models (document, relational, and graph) are widely used today, and each is
good in its respective domain. One model can be emulated in terms of another model
—for example, graph data can be represented in a relational database—but the result
is often awkward. That’s why we have different systems for different purposes, not a
single one-size-fits-all solution.

One thing that document and graph databases have in common is that they typically
don’t enforce a schema for the data they store, which can make it easier to adapt
applications to changing requirements. However, your application most likely still
assumes that data has a certain structure; it’s just a question of whether the schema is
explicit (enforced on write) or implicit (handled on read).

Each data model comes with its own query language or framework, and we discussed
several examples: SQL, MapReduce, MongoDB’s aggregation pipeline, Cypher,
SPARQL, and Datalog. We also touched on CSS and XSL/XPath, which aren’t data‐
base query languages but have interesting parallels.

Although we have covered a lot of ground, there are still many data models left
unmentioned. To give just a few brief examples:

• Researchers working with genome data often need to perform sequence-
similarity searches, which means taking one very long string (representing a
DNA molecule) and matching it against a large database of strings that are simi‐
lar, but not identical. None of the databases described here can handle this kind
of usage, which is why researchers have written specialized genome database
software like GenBank [48].

• Particle physicists have been doing Big Data–style large-scale data analysis for
decades, and projects like the Large Hadron Collider (LHC) now work with hun‐
dreds of petabytes! At such a scale custom solutions are required to stop the
hardware cost from spiraling out of control [49].

• Full-text search is arguably a kind of data model that is frequently used alongside
databases. Information retrieval is a large specialist subject that we won’t cover in
great detail in this book, but we’ll touch on search indexes in Chapter 3 and
Part III.

# Chapter 3: Storage and Retrieval

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

# Chapter 4: Encoding and Evolution

In this chapter we looked at several ways of turning data structures into bytes on the
network or bytes on disk. We saw how the details of these encodings affect not only
their efficiency, but more importantly also the architecture of applications and your
options for deploying them.

In particular, many services need to support rolling upgrades, where a new version of
a service is gradually deployed to a few nodes at a time, rather than deploying to all
nodes simultaneously. Rolling upgrades allow new versions of a service to be released
without downtime (thus encouraging frequent small releases over rare big releases)
and make deployments less risky (allowing faulty releases to be detected and rolled
back before they affect a large number of users). These properties are hugely benefi‐
cial for evolvability, the ease of making changes to an application.

During rolling upgrades, or for various other reasons, we must assume that different
nodes are running the different versions of our application’s code. Thus, it is impor‐
tant that all data flowing around the system is encoded in a way that provides back‐
ward compatibility (new code can read old data) and forward compatibility (old code
can read new data).

We discussed several data encoding formats and their compatibility properties:  

Programming language–specific encodings are restricted to a single program‐
ming language and often fail to provide forward and backward compatibility.

• Textual formats like JSON, XML, and CSV are widespread, and their compatibil‐
ity depends on how you use them. They have optional schema languages, which
are sometimes helpful and sometimes a hindrance. These formats are somewhat
vague about datatypes, so you have to be careful with things like numbers and
binary strings.

• Binary schema–driven formats like Thrift, Protocol Buffers, and Avro allow
compact, efficient encoding with clearly defined forward and backward compati‐
bility semantics. The schemas can be useful for documentation and code genera‐
tion in statically typed languages. However, they have the downside that data
needs to be decoded before it is human-readable.
We also discussed several modes of dataflow, illustrating different scenarios in which
data encodings are important:

• Databases, where the process writing to the database encodes the data and the
process reading from the database decodes it

• RPC and REST APIs, where the client encodes a request, the server decodes the
request and encodes a response, and the client finally decodes the response

• Asynchronous message passing (using message brokers or actors), where nodes
communicate by sending each other messages that are encoded by the sender
and decoded by the recipient

We can conclude that with a bit of care, backward/forward compatibility and rolling
upgrades are quite achievable. May your application’s evolution be rapid and your
deployments be frequent.

# Part II: Distributed Data

There are various reasons why you might want to distribute a database across multi‐
ple machines:

Scalability
If your data volume, read load, or write load grows bigger than a single machine
can handle, you can potentially spread the load across multiple machines.

Fault tolerance/high availability
If your application needs to continue working even if one machine (or several
machines, or the network, or an entire datacenter) goes down, you can use multi‐
ple machines to give you redundancy. When one fails, another one can take over.

Latency
If you have users around the world, you might want to have servers at various
locations worldwide so that each user can be served from a datacenter that is geo‐
graphically close to them. That avoids the users having to wait for network pack‐
ets to travel halfway around the world.

## Scaling to Higher Load

If all you need is to scale to higher load, the simplest approach is to buy a more pow‐
erful machine (sometimes called vertical scaling or scaling up). Many CPUs, many
RAM chips, and many disks can be joined together under one operating system, and
a fast interconnect allows any CPU to access any part of the memory or disk. In this
kind of shared-memory architecture, all the components can be treated as a single
machine [1]. i

The problem with a shared-memory approach is that the cost grows faster than line‐
arly: a machine with twice as many CPUs, twice as much RAM, and twice as much
disk capacity as another typically costs significantly more than twice as much. And
due to bottlenecks, a machine twice the size cannot necessarily handle twice the load.
A shared-memory architecture may offer limited fault tolerance—high-end machines
have hot-swappable components (you can replace disks, memory modules, and even
CPUs without shutting down the machines)—but it is definitely limited to a single
geographic location.

Another approach is the shared-disk architecture, which uses several machines with
independent CPUs and RAM, but stores data on an array of disks that is shared
between the machines, which are connected via a fast network. ii This architecture is
used for some data warehousing workloads, but contention and the overhead of lock‐
ing limit the scalability of the shared-disk approach [2].

## Shared-Nothing Architectures

By contrast, shared-nothing architectures [3] (sometimes called horizontal scaling or
scaling out) have gained a lot of popularity. In this approach, each machine or virtual
machine running the database software is called a node. Each node uses its CPUs,
RAM, and disks independently. Any coordination between nodes is done at the soft‐
ware level, using a conventional network.

No special hardware is required by a shared-nothing system, so you can use whatever
machines have the best price/performance ratio. You can potentially distribute data
across multiple geographic regions, and thus reduce latency for users and potentially
be able to survive the loss of an entire datacenter. With cloud deployments of virtual
machines, you don’t need to be operating at Google scale: even for small companies,
a multi-region distributed architecture is now feasible.

In this part of the book, we focus on shared-nothing architectures—not because they
are necessarily the best choice for every use case, but rather because they require the
most caution from you, the application developer. If your data is distributed across
multiple nodes, you need to be aware of the constraints and trade-offs that occur in
such a distributed system—the database cannot magically hide these from you.

While a distributed shared-nothing architecture has many advantages, it usually also
incurs additional complexity for applications and sometimes limits the expressive‐
ness of the data models you can use. In some cases, a simple single-threaded program
can perform significantly better than a cluster with over 100 CPU cores [4]. On the
other hand, shared-nothing systems can be very powerful. The next few chapters go
into details on the issues that arise when data is distributed.

## Replication Versus Partitioning

There are two common ways data is distributed across multiple nodes:

Replication
Keeping a copy of the same data on several different nodes, potentially in differ‐
ent locations. Replication provides redundancy: if some nodes are unavailable,
the data can still be served from the remaining nodes. Replication can also help
improve performance. We discuss replication in Chapter 5.

Partitioning
Splitting a big database into smaller subsets called partitions so that different par‐
titions can be assigned to different nodes (also known as sharding). We discuss
partitioning in Chapter 6.

These are separate mechanisms, but they often go hand in hand, as illustrated in
Figure II-1.

# Chapter 5:  Replication

