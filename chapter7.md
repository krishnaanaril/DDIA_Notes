# Summary

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

Dirty reads
One client reads another client’s writes before they have been committed. The
read committed isolation level and stronger levels prevent dirty reads.

Dirty writes
One client overwrites data that another client has written, but not yet committed.
Almost all transaction implementations prevent dirty writes.

Read skew (nonrepeatable reads)
A client sees different parts of the database at different points in time. This issue
is most commonly prevented with snapshot isolation, which allows a transaction
to read from a consistent snapshot at one point in time. It is usually implemented
with multi-version concurrency control (MVCC).

Lost updates
Two clients concurrently perform a read-modify-write cycle. One overwrites the
other’s write without incorporating its changes, so data is lost. Some implemen‐
tations of snapshot isolation prevent this anomaly automatically, while others
require a manual lock ( SELECT FOR UPDATE ).

Write skew
A transaction reads something, makes a decision based on the value it saw, and
writes the decision to the database. However, by the time the write is made, the
premise of the decision is no longer true. Only serializable isolation prevents this
anomaly.

Phantom reads
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

Two-phase locking
For decades this has been the standard way of implementing serializability, but
many applications avoid using it because of its performance characteristics.

Serializable snapshot isolation (SSI)
A fairly new algorithm that avoids most of the downsides of the previous
approaches. It uses an optimistic approach, allowing transactions to proceed
without blocking. When a transaction wants to commit, it is checked, and it is
aborted if the execution was not serializable.

The examples in this chapter used a relational data model. However, as discussed in
“The need for multi-object transactions” on page 231, transactions are a valuable
database feature, no matter which data model is used.