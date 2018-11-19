# Summary 

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