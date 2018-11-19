# Summary

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
Summary
|
63DNA molecule) and matching it against a large database of strings that are simi‐
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