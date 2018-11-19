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

1. Researchers working with genome data often need to perform sequence-
similarity searches, which means taking one very long string (representing a
DNA molecule) and matching it against a large database of strings that are simi‐
lar, but not identical. None of the databases described here can handle this kind
of usage, which is why researchers have written specialized genome database
software like GenBank [48].

2. Particle physicists have been doing Big Data–style large-scale data analysis for
decades, and projects like the Large Hadron Collider (LHC) now work with hun‐
dreds of petabytes! At such a scale custom solutions are required to stop the
hardware cost from spiraling out of control [49].

3. Full-text search is arguably a kind of data model that is frequently used alongside
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

1. OLTP systems are typically user-facing, which means that they may see a huge
volume of requests. In order to handle the load, applications usually only touch a
small number of records in each query. The application requests records using
some kind of key, and the storage engine uses an index to find the data for the
requested key. Disk seek time is often the bottleneck here.

2. Data warehouses and similar analytic systems are less well known, because they
are primarily used by business analysts, not by end users. They handle a much
lower volume of queries than OLTP systems, but each query is typically very
demanding, requiring many millions of records to be scanned in a short time.
Disk bandwidth (not seek time) is often the bottleneck here, and column-
oriented storage is an increasingly popular solution for this kind of workload.
On the OLTP side, we saw storage engines from two main schools of thought:

3. The log-structured school, which only permits appending to files and deleting
obsolete files, but never updates a file that has been written. Bitcask, SSTables,
LSM-trees, LevelDB, Cassandra, HBase, Lucene, and others belong to this group.

4. The update-in-place school, which treats the disk as a set of fixed-size pages that
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

1. Textual formats like JSON, XML, and CSV are widespread, and their compatibil‐
ity depends on how you use them. They have optional schema languages, which
are sometimes helpful and sometimes a hindrance. These formats are somewhat
vague about datatypes, so you have to be careful with things like numbers and
binary strings.

2. Binary schema–driven formats like Thrift, Protocol Buffers, and Avro allow
compact, efficient encoding with clearly defined forward and backward compati‐
bility semantics. The schemas can be useful for documentation and code genera‐
tion in statically typed languages. However, they have the downside that data
needs to be decoded before it is human-readable.
We also discussed several modes of dataflow, illustrating different scenarios in which
data encodings are important:

3. Databases, where the process writing to the database encodes the data and the
process reading from the database decodes it

4. RPC and REST APIs, where the client encodes a request, the server decodes the
request and encodes a response, and the client finally decodes the response

5. Asynchronous message passing (using message brokers or actors), where nodes
communicate by sending each other messages that are encoded by the sender
and decoded by the recipient

We can conclude that with a bit of care, backward/forward compatibility and rolling
upgrades are quite achievable. May your application’s evolution be rapid and your
deployments be frequent.

# Part II: Distributed Data

There are various reasons why you might want to distribute a database across multi‐
ple machines:

### Scalability

If your data volume, read load, or write load grows bigger than a single machine
can handle, you can potentially spread the load across multiple machines.

### Fault tolerance/high availability

If your application needs to continue working even if one machine (or several
machines, or the network, or an entire datacenter) goes down, you can use multi‐
ple machines to give you redundancy. When one fails, another one can take over.

### Latency

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

### Replication
Keeping a copy of the same data on several different nodes, potentially in differ‐
ent locations. Replication provides redundancy: if some nodes are unavailable,
the data can still be served from the remaining nodes. Replication can also help
improve performance. We discuss replication in Chapter 5.

### Partitioning
Splitting a big database into smaller subsets called partitions so that different par‐
titions can be assigned to different nodes (also known as sharding). We discuss
partitioning in Chapter 6.

These are separate mechanisms, but they often go hand in hand, as illustrated in
Figure II-1.

# Chapter 5:  Replication

In this chapter we looked at the issue of replication. Replication can serve several
purposes:

### High availability
Keeping the system running, even when one machine (or several machines, or an
entire datacenter) goes down

### Disconnected operation
Allowing an application to continue working when there is a network interrup‐
tion

### Latency
Placing data geographically close to users, so that users can interact with it faster

### Scalability
Being able to handle a higher volume of reads than a single machine could han‐
dle, by performing reads on replicas

Despite being a simple goal—keeping a copy of the same data on several machines—
replication turns out to be a remarkably tricky problem. It requires carefully thinking
about concurrency and about all the things that can go wrong, and dealing with the
consequences of those faults. At a minimum, we need to deal with unavailable nodes
and network interruptions (and that’s not even considering the more insidious kinds
of fault, such as silent data corruption due to software bugs).

We discussed three main approaches to replication:

### Single-leader replication
Clients send all writes to a single node (the leader), which sends a stream of data
change events to the other replicas (followers). Reads can be performed on any
replica, but reads from followers might be stale.

### Multi-leader replication
Clients send each write to one of several leader nodes, any of which can accept
writes. The leaders send streams of data change events to each other and to any
follower nodes.

### Leaderless replication
Clients send each write to several nodes, and read from several nodes in parallel
in order to detect and correct nodes with stale data.

Each approach has advantages and disadvantages. Single-leader replication is popular
because it is fairly easy to understand and there is no conflict resolution to worry
about. Multi-leader and leaderless replication can be more robust in the presence of
faulty nodes, network interruptions, and latency spikes—at the cost of being harder
to reason about and providing only very weak consistency guarantees.

Replication can be synchronous or asynchronous, which has a profound effect on the
system behavior when there is a fault. Although asynchronous replication can be fast
when the system is running smoothly, it’s important to figure out what happens
when replication lag increases and servers fail. If a leader fails and you promote an
asynchronously updated follower to be the new leader, recently committed data may
be lost.

We looked at some strange effects that can be caused by replication lag, and we dis‐
cussed a few consistency models which are helpful for deciding how an application
should behave under replication lag:

### Read-after-write consistency
Users should always see data that they submitted themselves.

### Monotonic reads
After users have seen the data at one point in time, they shouldn’t later see the
data from some earlier point in time.

### Consistent prefix reads
Users should see the data in a state that makes causal sense: for example, seeing a
question and its reply in the correct order.

Finally, we discussed the concurrency issues that are inherent in multi-leader and
leaderless replication approaches: because they allow multiple writes to happen con‐
currently, conflicts may occur. We examined an algorithm that a database might use
to determine whether one operation happened before another, or whether they hap‐
pened concurrently. We also touched on methods for resolving conflicts by merging
together concurrent updates.

# Chapter 6: Partitioning

In this chapter we explored different ways of partitioning a large dataset into smaller
subsets. Partitioning is necessary when you have so much data that storing and pro‐
cessing it on a single machine is no longer feasible.

The goal of partitioning is to spread the data and query load evenly across multiple
machines, avoiding hot spots (nodes with disproportionately high load). This
requires choosing a partitioning scheme that is appropriate to your data, and reba‐
lancing the partitions when nodes are added to or removed from the cluster.
We discussed two main approaches to partitioning:

1. Key range partitioning, where keys are sorted, and a partition owns all the keys
from some minimum up to some maximum. Sorting has the advantage that effi‐
cient range queries are possible, but there is a risk of hot spots if the application
often accesses keys that are close together in the sorted order.
In this approach, partitions are typically rebalanced dynamically by splitting the
range into two subranges when a partition gets too big.

2. Hash partitioning, where a hash function is applied to each key, and a partition
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

1. Document-partitioned indexes (local indexes), where the secondary indexes are
stored in the same partition as the primary key and value. This means that only a
single partition needs to be updated on write, but a read of the secondary index
requires a scatter/gather across all partitions.

2. Term-partitioned indexes (global indexes), where the secondary indexes are parti‐
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

# Chapter 7: Transactions

Transactions are an abstraction layer that allows an application to pretend that cer‐
tain concurrency problems and certain kinds of hardware and software faults don’t
exist. A large class of errors is reduced down to a simple transaction abort, and the
application just needs to try again.

In this chapter we saw many examples of problems that transactions help prevent.
Not all applications are susceptible to all those problems: an application with very
simple access patterns, such as reading and writing only a single record, can probably
manage without transactions. However, for more complex access patterns, transac‐
tions can hugely reduce the number of potential error cases you need to think about.

Without transactions, various error scenarios (processes crashing, network interrup‐
tions, power outages, disk full, unexpected concurrency, etc.) mean that data can
become inconsistent in various ways. For example, denormalized data can easily go
out of sync with the source data. Without transactions, it becomes very difficult to
reason about the effects that complex interacting accesses can have on the database.

In this chapter, we went particularly deep into the topic of concurrency control. We
discussed several widely used isolation levels, in particular read committed, snapshot
isolation (sometimes called repeatable read), and serializable. We characterized those
isolation levels by discussing various examples of race conditions:

### Dirty reads
One client reads another client’s writes before they have been committed. The
read committed isolation level and stronger levels prevent dirty reads.

### Dirty writes
One client overwrites data that another client has written, but not yet committed.
Almost all transaction implementations prevent dirty writes.

### Read skew (nonrepeatable reads)
A client sees different parts of the database at different points in time. This issue
is most commonly prevented with snapshot isolation, which allows a transaction
to read from a consistent snapshot at one point in time. It is usually implemented
with multi-version concurrency control (MVCC).

### Lost updates
Two clients concurrently perform a read-modify-write cycle. One overwrites the
other’s write without incorporating its changes, so data is lost. Some implemen‐
tations of snapshot isolation prevent this anomaly automatically, while others
require a manual lock ( SELECT FOR UPDATE ).

### Write skew
A transaction reads something, makes a decision based on the value it saw, and
writes the decision to the database. However, by the time the write is made, the
premise of the decision is no longer true. Only serializable isolation prevents this
anomaly.

### Phantom reads
A transaction reads objects that match some search condition. Another client
makes a write that affects the results of that search. Snapshot isolation prevents
straightforward phantom reads, but phantoms in the context of write skew
require special treatment, such as index-range locks.

Weak isolation levels protect against some of those anomalies but leave you, the
application developer, to handle others manually (e.g., using explicit locking). Only
serializable isolation protects against all of these issues. We discussed three different
approaches to implementing serializable transactions:

Literally executing transactions in a serial order
If you can make each transaction very fast to execute, and the transaction
throughput is low enough to process on a single CPU core, this is a simple and
effective option.

### Two-phase locking
For decades this has been the standard way of implementing serializability, but
many applications avoid using it because of its performance characteristics.

### Serializable snapshot isolation (SSI)
A fairly new algorithm that avoids most of the downsides of the previous
approaches. It uses an optimistic approach, allowing transactions to proceed
without blocking. When a transaction wants to commit, it is checked, and it is
aborted if the execution was not serializable.

The examples in this chapter used a relational data model. However, as discussed in
“The need for multi-object transactions” on page 231, transactions are a valuable
database feature, no matter which data model is used.

# Chapter 8: The Trouble with Distributed Systems

In this chapter we have discussed a wide range of problems that can occur in dis‐
tributed systems, including:

1. Whenever you try to send a packet over the network, it may be lost or arbitrarily
delayed. Likewise, the reply may be lost or delayed, so if you don’t get a reply,
you have no idea whether the message got through.

2. A node’s clock may be significantly out of sync with other nodes (despite your
best efforts to set up NTP), it may suddenly jump forward or back in time, and
relying on it is dangerous because you most likely don’t have a good measure of
your clock’s error interval.

3. A process may pause for a substantial amount of time at any point in its execu‐
tion (perhaps due to a stop-the-world garbage collector), be declared dead by
other nodes, and then come back to life again without realizing that it was
paused.

The fact that such partial failures can occur is the defining characteristic of dis‐
tributed systems. Whenever software tries to do anything involving other nodes,
there is the possibility that it may occasionally fail, or randomly go slow, or not
respond at all (and eventually time out). In distributed systems, we try to build toler‐
ance of partial failures into software, so that the system as a whole may continue
functioning even when some of its constituent parts are broken.

To tolerate faults, the first step is to detect them, but even that is hard. Most systems
don’t have an accurate mechanism of detecting whether a node has failed, so most
distributed algorithms rely on timeouts to determine whether a remote node is still
available. However, timeouts can’t distinguish between network and node failures,
and variable network delay sometimes causes a node to be falsely suspected of crash‐
ing. Moreover, sometimes a node can be in a degraded state: for example, a Gigabit
network interface could suddenly drop to 1 Kb/s throughput due to a driver bug [94].
Such a node that is “limping” but not dead can be even more difficult to deal with
than a cleanly failed node.

Once a fault is detected, making a system tolerate it is not easy either: there is no
global variable, no shared memory, no common knowledge or any other kind of
shared state between the machines. Nodes can’t even agree on what time it is, let
alone on anything more profound. The only way information can flow from one
node to another is by sending it over the unreliable network. Major decisions cannot
be safely made by a single node, so we require protocols that enlist help from other
nodes and try to get a quorum to agree.

If you’re used to writing software in the idealized mathematical perfection of a single
computer, where the same operation always deterministically returns the same result,
then moving to the messy physical reality of distributed systems can be a bit of a
shock. Conversely, distributed systems engineers will often regard a problem as triv‐
ial if it can be solved on a single computer [5], and indeed a single computer can do a
lot nowadays [95]. If you can avoid opening Pandora’s box and simply keep things on
a single machine, it is generally worth doing so.

However, as discussed in the introduction to Part II, scalability is not the only reason
for wanting to use a distributed system. Fault tolerance and low latency (by placing
data geographically close to users) are equally important goals, and those things can‐
not be achieved with a single node.

In this chapter we also went on some tangents to explore whether the unreliability of
networks, clocks, and processes is an inevitable law of nature. We saw that it isn’t: it
is possible to give hard real-time response guarantees and bounded delays in net‐
works, but doing so is very expensive and results in lower utilization of hardware
resources. Most non-safety-critical systems choose cheap and unreliable over expen‐
sive and reliable.

We also touched on supercomputers, which assume reliable components and thus
have to be stopped and restarted entirely when a component does fail. By contrast,
distributed systems can run forever without being interrupted at the service level,
because all faults and maintenance can be handled at the node level—at least in
theory. (In practice, if a bad configuration change is rolled out to all nodes, that will
still bring a distributed system to its knees.)

# Chapter 9: Consistency and Consensus

In this chapter we examined the topics of consistency and consensus from several dif‐
ferent angles. We looked in depth at linearizability, a popular consistency model: its
goal is to make replicated data appear as though there were only a single copy, and to
make all operations act on it atomically. Although linearizability is appealing because
it is easy to understand—it makes a database behave like a variable in a single-
threaded program—it has the downside of being slow, especially in environments
with large network delays.

We also explored causality, which imposes an ordering on events in a system (what
happened before what, based on cause and effect). Unlike linearizability, which puts
all operations in a single, totally ordered timeline, causality provides us with a weaker
consistency model: some things can be concurrent, so the version history is like a
timeline with branching and merging. Causal consistency does not have the coordi‐
nation overhead of linearizability and is much less sensitive to network problems.

However, even if we capture the causal ordering (for example using Lamport time‐
stamps), we saw that some things cannot be implemented this way: in “Timestamp
ordering is not sufficient” on page 347 we considered the example of ensuring that a
username is unique and rejecting concurrent registrations for the same username. If
one node is going to accept a registration, it needs to somehow know that another
node isn’t concurrently in the process of registering the same name. This problem led
us toward consensus.

We saw that achieving consensus means deciding something in such a way that all
nodes agree on what was decided, and such that the decision is irrevocable. With
some digging, it turns out that a wide range of problems are actually reducible to
consensus and are equivalent to each other (in the sense that if you have a solution
for one of them, you can easily transform it into a solution for one of the others).
Such equivalent problems include:

### Linearizable compare-and-set registers
The register needs to atomically decide whether to set its value, based on whether
its current value equals the parameter given in the operation.

### Atomic transaction commit
A database must decide whether to commit or abort a distributed transaction.

### Total order broadcast
The messaging system must decide on the order in which to deliver messages.

### Locks and leases
When several clients are racing to grab a lock or lease, the lock decides which one
successfully acquired it.

### Membership/coordination service
Given a failure detector (e.g., timeouts), the system must decide which nodes are
alive, and which should be considered dead because their sessions timed out.

### Uniqueness constraint
When several transactions concurrently try to create conflicting records with the
same key, the constraint must decide which one to allow and which should fail
with a constraint violation.

All of these are straightforward if you only have a single node, or if you are willing to
assign the decision-making capability to a single node. This is what happens in a
single-leader database: all the power to make decisions is vested in the leader, which
is why such databases are able to provide linearizable operations, uniqueness con‐
straints, a totally ordered replication log, and more.

However, if that single leader fails, or if a network interruption makes the leader
unreachable, such a system becomes unable to make any progress. There are three
ways of handling that situation:

1. Wait for the leader to recover, and accept that the system will be blocked in the
meantime. Many XA/JTA transaction coordinators choose this option. This
approach does not fully solve consensus because it does not satisfy the termina‐
tion property: if the leader does not recover, the system can be blocked forever.

2. Manually fail over by getting humans to choose a new leader node and reconfig‐
ure the system to use it. Many relational databases take this approach. It is a kind
of consensus by “act of God”—the human operator, outside of the computer sys‐
tem, makes the decision. The speed of failover is limited by the speed at which
humans can act, which is generally slower than computers.

3. Use an algorithm to automatically choose a new leader. This approach requires a
consensus algorithm, and it is advisable to use a proven algorithm that correctly
handles adverse network conditions [107].

Although a single-leader database can provide linearizability without executing a
consensus algorithm on every write, it still requires consensus to maintain its leader‐
ship and for leadership changes. Thus, in some sense, having a leader only “kicks the
can down the road”: consensus is still required, only in a different place, and less fre‐
quently. The good news is that fault-tolerant algorithms and systems for consensus
exist, and we briefly discussed them in this chapter.

Tools like ZooKeeper play an important role in providing an “outsourced” consen‐
sus, failure detection, and membership service that applications can use. It’s not easy
to use, but it is much better than trying to develop your own algorithms that can
withstand all the problems discussed in Chapter 8. If you find yourself wanting to do
one of those things that is reducible to consensus, and you want it to be fault-tolerant,
then it is advisable to use something like ZooKeeper.

Nevertheless, not every system necessarily requires consensus: for example, leaderless
and multi-leader replication systems typically do not use global consensus. The con‐
flicts that occur in these systems (see “Handling Write Conflicts” on page 171) are a
consequence of not having consensus across different leaders, but maybe that’s okay:
maybe we simply need to cope without linearizability and learn to work better with
data that has branching and merging version histories.

This chapter referenced a large body of research on the theory of distributed systems.
Although the theoretical papers and proofs are not always easy to understand, and
sometimes make unrealistic assumptions, they are incredibly valuable for informing
practical work in this field: they help us reason about what can and cannot be done,
and help us find the counterintuitive ways in which distributed systems are often
flawed. If you have the time, the references are well worth exploring.

# Part III: Derived Data

In Parts I and II of this book, we assembled from the ground up all the major consid‐
erations that go into a distributed database, from the layout of data on disk all the
way to the limits of distributed consistency in the presence of faults. However, this
discussion assumed that there was only one database in the application.

In reality, data systems are often more complex. In a large application you often need
to be able to access and process data in many different ways, and there is no one data‐
base that can satisfy all those different needs simultaneously. Applications thus com‐
monly use a combination of several different datastores, indexes, caches, analytics
systems, etc. and implement mechanisms for moving data from one store to another.

In this final part of the book, we will examine the issues around integrating multiple
different data systems, potentially with different data models and optimized for dif‐
ferent access patterns, into one coherent application architecture. This aspect of
system-building is often overlooked by vendors who claim that their product can sat‐
isfy all your needs. In reality, integrating disparate systems is one of the most impor‐
tant things that needs to be done in a nontrivial application.

## Systems of Record and Derived Data

On a high level, systems that store and process data can be grouped into two broad
categories:

### Systems of record
A system of record, also known as source of truth, holds the authoritative version
of your data. When new data comes in, e.g., as user input, it is first written here.
Each fact is represented exactly once (the representation is typically normalized).
If there is any discrepancy between another system and the system of record,
then the value in the system of record is (by definition) the correct one.

### Derived data systems
Data in a derived system is the result of taking some existing data from another
system and transforming or processing it in some way. If you lose derived data,
you can recreate it from the original source. A classic example is a cache: data can
be served from the cache if present, but if the cache doesn’t contain what you
need, you can fall back to the underlying database. Denormalized values, indexes,
and materialized views also fall into this category. In recommendation systems,
predictive summary data is often derived from usage logs.

Technically speaking, derived data is redundant, in the sense that it duplicates exist‐
ing information. However, it is often essential for getting good performance on read
queries. It is commonly denormalized. You can derive several different datasets from
a single source, enabling you to look at the data from different “points of view.”
Not all systems make a clear distinction between systems of record and derived data
in their architecture, but it’s a very helpful distinction to make, because it clarifies the dataflow through your system: it makes explicit which parts of the system have which
inputs and which outputs, and how they depend on each other.

Most databases, storage engines, and query languages are not inherently either a sys‐
tem of record or a derived system. A database is just a tool: how you use it is up to
you. The distinction between system of record and derived data system depends not
on the tool, but on how you use it in your application.

By being clear about which data is derived from which other data, you can bring
clarity to an otherwise confusing system architecture. This point will be a running
theme throughout this part of the book.

# Chapter 10: Batch Processing

In this chapter we explored the topic of batch processing. We started by looking at
Unix tools such as awk , grep , and sort , and we saw how the design philosophy of
those tools is carried forward into MapReduce and more recent dataflow engines.
Some of those design principles are that inputs are immutable, outputs are intended
to become the input to another (as yet unknown) program, and complex problems
are solved by composing small tools that “do one thing well.”

In the Unix world, the uniform interface that allows one program to be composed
with another is files and pipes; in MapReduce, that interface is a distributed filesys‐
tem. We saw that dataflow engines add their own pipe-like data transport mecha‐
nisms to avoid materializing intermediate state to the distributed filesystem, but the
initial input and final output of a job is still usually HDFS.

The two main problems that distributed batch processing frameworks need to solve
are:

### Partitioning
In MapReduce, mappers are partitioned according to input file blocks. The out‐
put of mappers is repartitioned, sorted, and merged into a configurable number
of reducer partitions. The purpose of this process is to bring all the related data—
e.g., all the records with the same key—together in the same place.
Post-MapReduce dataflow engines try to avoid sorting unless it is required, but
they otherwise take a broadly similar approach to partitioning.

### Fault tolerance
MapReduce frequently writes to disk, which makes it easy to recover from an
individual failed task without restarting the entire job but slows down execution
in the failure-free case. Dataflow engines perform less materialization of inter‐
mediate state and keep more in memory, which means that they need to recom‐
pute more data if a node fails. Deterministic operators reduce the amount of data
that needs to be recomputed.

We discussed several join algorithms for MapReduce, most of which are also inter‐
nally used in MPP databases and dataflow engines. They also provide a good illustra‐
tion of how partitioned algorithms work:

### Sort-merge joins
Each of the inputs being joined goes through a mapper that extracts the join key.
By partitioning, sorting, and merging, all the records with the same key end up
going to the same call of the reducer. This function can then output the joined
records.

### Broadcast hash joins
One of the two join inputs is small, so it is not partitioned and it can be entirely
loaded into a hash table. Thus, you can start a mapper for each partition of the
large join input, load the hash table for the small input into each mapper, and
then scan over the large input one record at a time, querying the hash table for
each record.

### Partitioned hash joins
If the two join inputs are partitioned in the same way (using the same key, same
hash function, and same number of partitions), then the hash table approach can
be used independently for each partition.

Distributed batch processing engines have a deliberately restricted programming
model: callback functions (such as mappers and reducers) are assumed to be stateless
and to have no externally visible side effects besides their designated output. This
restriction allows the framework to hide some of the hard distributed systems prob‐
lems behind its abstraction: in the face of crashes and network issues, tasks can be
retried safely, and the output from any failed tasks is discarded. If several tasks for a
partition succeed, only one of them actually makes its output visible.

Thanks to the framework, your code in a batch processing job does not need to worry
about implementing fault-tolerance mechanisms: the framework can guarantee that
the final output of a job is the same as if no faults had occurred, even though in real‐
ity various tasks perhaps had to be retried. These reliable semantics are much stron‐
ger than what you usually have in online services that handle user requests and that
write to databases as a side effect of processing a request.

The distinguishing feature of a batch processing job is that it reads some input data
and produces some output data, without modifying the input—in other words, the
output is derived from the input. Crucially, the input data is bounded: it has a known,
fixed size (for example, it consists of a set of log files at some point in time, or a snap‐shot of a database’s contents). Because it is bounded, a job knows when it has finished reading the entire input, and so a job eventually completes when it is done.

In the next chapter, we will turn to stream processing, in which the input is unboun‐
ded—that is, you still have a job, but its inputs are never-ending streams of data. In
this case, a job is never complete, because at any time there may still be more work
coming in. We shall see that stream and batch processing are similar in some
respects, but the assumption of unbounded streams also changes a lot about how we
build systems.

# Chapter 11: Stream Processing

In this chapter we have discussed event streams, what purposes they serve, and how
to process them. In some ways, stream processing is very much like the batch pro‐
cessing we discussed in Chapter 10, but done continuously on unbounded (never-
ending) streams rather than on a fixed-size input. From this perspective, message
brokers and event logs serve as the streaming equivalent of a filesystem.

We spent some time comparing two types of message brokers:

### AMQP/JMS-style message broker
The broker assigns individual messages to consumers, and consumers acknowl‐
edge individual messages when they have been successfully processed. Messages
are deleted from the broker once they have been acknowledged. This approach is
appropriate as an asynchronous form of RPC (see also “Message-Passing Data‐
flow” on page 136), for example in a task queue, where the exact order of mes‐
sage processing is not important and where there is no need to go back and read
old messages again after they have been processed.

### Log-based message broker
The broker assigns all messages in a partition to the same consumer node, and
always delivers messages in the same order. Parallelism is achieved through par‐
titioning, and consumers track their progress by checkpointing the offset of the
last message they have processed. The broker retains messages on disk, so it is
possible to jump back and reread old messages if necessary.

The log-based approach has similarities to the replication logs found in databases
(see Chapter 5) and log-structured storage engines (see Chapter 3). We saw that this
approach is especially appropriate for stream processing systems that consume input
streams and generate derived state or derived output streams.

In terms of where streams come from, we discussed several possibilities: user activity
events, sensors providing periodic readings, and data feeds (e.g., market data in
finance) are naturally represented as streams. We saw that it can also be useful to
think of the writes to a database as a stream: we can capture the changelog—i.e., the
history of all changes made to a database—either implicitly through change data cap‐
ture or explicitly through event sourcing. Log compaction allows the stream to retain
a full copy of the contents of a database.

Representing databases as streams opens up powerful opportunities for integrating
systems. You can keep derived data systems such as search indexes, caches, and ana‐
lytics systems continually up to date by consuming the log of changes and applying
them to the derived system. You can even build fresh views onto existing data by
starting from scratch and consuming the log of changes from the beginning all the
way to the present.

The facilities for maintaining state as streams and replaying messages are also the
basis for the techniques that enable stream joins and fault tolerance in various stream
processing frameworks. We discussed several purposes of stream processing, includ‐
ing searching for event patterns (complex event processing), computing windowed
aggregations (stream analytics), and keeping derived data systems up to date (materi‐
alized views).

We then discussed the difficulties of reasoning about time in a stream processor,
including the distinction between processing time and event timestamps, and the
problem of dealing with straggler events that arrive after you thought your window
was complete.

We distinguished three types of joins that may appear in stream processes:

### Stream-stream joins
Both input streams consist of activity events, and the join operator searches for
related events that occur within some window of time. For example, it may
match two actions taken by the same user within 30 minutes of each other. The
two join inputs may in fact be the same stream (a self-join) if you want to find
related events within that one stream.

### Stream-table joins
One input stream consists of activity events, while the other is a database change‐
log. The changelog keeps a local copy of the database up to date. For each activity
event, the join operator queries the database and outputs an enriched activity
event.

### Table-table joins
Both input streams are database changelogs. In this case, every change on one
side is joined with the latest state of the other side. The result is a stream of
changes to the materialized view of the join between the two tables.

Finally, we discussed techniques for achieving fault tolerance and exactly-once
semantics in a stream processor. As with batch processing, we need to discard the
partial output of any failed tasks. However, since a stream process is long-running
and produces output continuously, we can’t simply discard all output. Instead, a
finer-grained recovery mechanism can be used, based on microbatching, checkpoint‐
ing, transactions, or idempotent writes.

# Chapter 12: The Future of Data Systems

In this chapter we discussed new approaches to designing data systems, and I
included my personal opinions and speculations about the future. We started with
the observation that there is no one single tool that can efficiently serve all possible
use cases, and so applications necessarily need to compose several different pieces of
software to accomplish their goals. We discussed how to solve this data integration
problem by using batch processing and event streams to let data changes flow
between different systems.

In this approach, certain systems are designated as systems of record, and other data
is derived from them through transformations. In this way we can maintain indexes,
materialized views, machine learning models, statistical summaries, and more. By
making these derivations and transformations asynchronous and loosely coupled, a
problem in one area is prevented from spreading to unrelated parts of the system,
increasing the robustness and fault-tolerance of the system as a whole.

Expressing dataflows as transformations from one dataset to another also helps
evolve applications: if you want to change one of the processing steps, for example to
change the structure of an index or cache, you can just rerun the new transformation
code on the whole input dataset in order to rederive the output. Similarly, if some‐
thing goes wrong, you can fix the code and reprocess the data in order to recover.

These processes are quite similar to what databases already do internally, so we recast
the idea of dataflow applications as unbundling the components of a database, and
building an application by composing these loosely coupled components.
Derived state can be updated by observing changes in the underlying data. Moreover,
the derived state itself can further be observed by downstream consumers. We can
even take this dataflow all the way through to the end-user device that is displaying
the data, and thus build user interfaces that dynamically update to reflect data
changes and continue to work offline.

Next, we discussed how to ensure that all of this processing remains correct in the
presence of faults. We saw that strong integrity guarantees can be implemented scala‐
bly with asynchronous event processing, by using end-to-end operation identifiers to
make operations idempotent and by checking constraints asynchronously. Clients
can either wait until the check has passed, or go ahead without waiting but risk hav‐
ing to apologize about a constraint violation. This approach is much more scalable
and robust than the traditional approach of using distributed transactions, and fits
with how many business processes work in practice.

By structuring applications around dataflow and checking constraints asynchro‐
nously, we can avoid most coordination and create systems that maintain integrity
but still perform well, even in geographically distributed scenarios and in the pres‐
ence of faults. We then talked a little about using audits to verify the integrity of data
and detect corruption.

Finally, we took a step back and examined some ethical aspects of building data-
intensive applications. We saw that although data can be used to do good, it can also
do significant harm: making justifying decisions that seriously affect people’s lives
and are difficult to appeal against, leading to discrimination and exploitation, nor‐
malizing surveillance, and exposing intimate information. We also run the risk of
data breaches, and we may find that a well-intentioned use of data has unintended
consequences.

As software and data are having such a large impact on the world, we engineers must
remember that we carry a responsibility to work toward the kind of world that we
want to live in: a world that treats people with humanity and respect. I hope that we
can work together toward that goal.
