# Summary

In this chapter we looked at the issue of replication. Replication can serve several
purposes:

High availability
Keeping the system running, even when one machine (or several machines, or an
entire datacenter) goes down

Disconnected operation
Allowing an application to continue working when there is a network interrup‐
tion

Latency
Placing data geographically close to users, so that users can interact with it faster

Scalability
Being able to handle a higher volume of reads than a single machine could han‐
dle, by performing reads on replicas

Despite being a simple goal—keeping a copy of the same data on several machines—
replication turns out to be a remarkably tricky problem. It requires carefully thinking
about concurrency and about all the things that can go wrong, and dealing with the
consequences of those faults. At a minimum, we need to deal with unavailable nodes
and network interruptions (and that’s not even considering the more insidious kinds
of fault, such as silent data corruption due to software bugs).

We discussed three main approaches to replication:

Single-leader replication
Clients send all writes to a single node (the leader), which sends a stream of data
change events to the other replicas (followers). Reads can be performed on any
replica, but reads from followers might be stale.

Multi-leader replication
Clients send each write to one of several leader nodes, any of which can accept
writes. The leaders send streams of data change events to each other and to any
follower nodes.

Leaderless replication
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

Read-after-write consistency
Users should always see data that they submitted themselves.

Monotonic reads
After users have seen the data at one point in time, they shouldn’t later see the
data from some earlier point in time.

Consistent prefix reads
Users should see the data in a state that makes causal sense: for example, seeing a
question and its reply in the correct order.

Finally, we discussed the concurrency issues that are inherent in multi-leader and
leaderless replication approaches: because they allow multiple writes to happen con‐
currently, conflicts may occur. We examined an algorithm that a database might use
to determine whether one operation happened before another, or whether they hap‐
pened concurrently. We also touched on methods for resolving conflicts by merging
together concurrent updates.