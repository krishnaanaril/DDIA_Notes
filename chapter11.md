# Summary

In this chapter we have discussed event streams, what purposes they serve, and how
to process them. In some ways, stream processing is very much like the batch pro‐
cessing we discussed in Chapter 10, but done continuously on unbounded (never-
ending) streams rather than on a fixed-size input. From this perspective, message
brokers and event logs serve as the streaming equivalent of a filesystem.

We spent some time comparing two types of message brokers:

AMQP/JMS-style message broker
The broker assigns individual messages to consumers, and consumers acknowl‐
edge individual messages when they have been successfully processed. Messages
are deleted from the broker once they have been acknowledged. This approach is
appropriate as an asynchronous form of RPC (see also “Message-Passing Data‐
flow” on page 136), for example in a task queue, where the exact order of mes‐
sage processing is not important and where there is no need to go back and read
old messages again after they have been processed.

Log-based message broker
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

Stream-stream joins
Both input streams consist of activity events, and the join operator searches for
related events that occur within some window of time. For example, it may
match two actions taken by the same user within 30 minutes of each other. The
two join inputs may in fact be the same stream (a self-join) if you want to find
related events within that one stream.

Stream-table joins
One input stream consists of activity events, while the other is a database change‐
log. The changelog keeps a local copy of the database up to date. For each activity
event, the join operator queries the database and outputs an enriched activity
event.

Table-table joins
Both input streams are database changelogs. In this case, every change on one
side is joined with the latest state of the other side. The result is a stream of
changes to the materialized view of the join between the two tables.

Finally, we discussed techniques for achieving fault tolerance and exactly-once
semantics in a stream processor. As with batch processing, we need to discard the
partial output of any failed tasks. However, since a stream process is long-running
and produces output continuously, we can’t simply discard all output. Instead, a
finer-grained recovery mechanism can be used, based on microbatching, checkpoint‐
ing, transactions, or idempotent writes.