# Summary

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