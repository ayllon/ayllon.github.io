---
title: Event access
---

## Outline

### Problem statement
As of today, LHC data is stored as ROOT[^root] files. Some of the analysis
on these files are relatively simple analytical queries, which as of
today are done with hand-written C++ programs. It has been already
proposed that declarative queries would likely be easier to write for physicists[^raw].

As far as I know, this approach never went beyond a proof-of-concept.

It could be interesting to allow physicist to query storages directly for the
set of events they are interested in, rather than having to do remote I/O
and/or downloading the whole file to their machine or computing node.

### Proposed solution
A declarative API running on a storage element (i.e. EOS[^eos]), so the users
could ask directly for the set of events they are interested in, rather than
having to write complex C++ programs that need to search file per file.

This solution should perform at least as good as a bare C++ program.

### Possible approaches
There has been prototyping efforts to use the CPU of the EOS disks servers for running
user jobs. Part of the system could follow the same approach, run
jobs on EOS disks, and then aggregate.

### Difficulties
* The system should not be limited to a single file format (ROOT), and much less
to a specific way of encoding event data on these files.
* How to allow for extra formats?
* A query language is not a trivial task, nor a query planner/optimizer.
  * Maybe there are reusable bits? [^presto]
* Probably, plenty of literature.
* Is it really different to an existing Cluster DB? Are ROOT files after all
Object Databases? Are we just re-doing RAW[^raw] or Hadoop?
* Is there a niche for middleware here? Something that sits between custom file
formats spread around on EOS (or whatever) and the query language? Sort of the clang
AST for distributed queries.

[^root]: [ROOT - An Object Oriented Data Analysis Framework](http://www-ai.cs.uni-dortmund.de/PublicPublicationFiles/brun_rademakers_97a_2.pdf), Rene Brun and Fons Rademakers
[^raw]: [Adaptive query processing on RAW data](http://dl.acm.org/citation.cfm?id=2732986), Manos Karpathiotakis et al
[^eos]: [Exabyte Scale Storage at CERN](http://iopscience.iop.org/article/10.1088/1742-6596/331/5/052015/meta), Andreas J Peters and Lukasz Janyst
[^presto]: [Presto](https://prestodb.io/)
