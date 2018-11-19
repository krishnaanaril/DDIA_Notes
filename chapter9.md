# Summary

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

Linearizable compare-and-set registers
The register needs to atomically decide whether to set its value, based on whether
its current value equals the parameter given in the operation.

Atomic transaction commit
A database must decide whether to commit or abort a distributed transaction.

Total order broadcast
The messaging system must decide on the order in which to deliver messages.

Locks and leases
When several clients are racing to grab a lock or lease, the lock decides which one
successfully acquired it.

Membership/coordination service
Given a failure detector (e.g., timeouts), the system must decide which nodes are
alive, and which should be considered dead because their sessions timed out.

Uniqueness constraint
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


