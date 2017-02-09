---
title: And about what?
---

## And about what?
So, obviously, the first task has to be decide the subject of the thesis.
On the plus side, since I am not doing the PhD with a grant, I have relative
freedom on the subject. At the same time, that can be a negative point, because
it makes harder to decide.

More or less the low resolution picture is there. Since I would rather be able
to spend work time on the thesis, I need to make it relevant to my daily job.
Which basically means, it has to do something with data transfer and/or data
management.

Luckily, we had an informal discussion - more of a brainstorm, really - around
December 2017 about how the Grid can look like for the
[High Luminosity LHC.](http://hilumilhc.web.cern.ch/). The amount of generated
data may outpace the growth of the available resources, specially network
and disk. Budget is flat, and that's being lucky, so the growth will depend
purely on the technology progress (e.g data density).

That will require changes on the approach to data replication and processing.
I will throw here just a bunch of words and ideas that have found around:

* Tape as only persistent replica (tapes are cheaper per byte), disks for cache
(Disks account for 50% of the budget)
* Change data granularity (i.e. from file to event)
* Credit sites per jobs run, not data stored
* Data access via WAN instead of replication
* Integration of cloud resources (this is already being worked on)
* P2P data access/replication
* Storage federation to reduce operation complexity
(i.e. one endpoint for all storages in Europe)
* Catalog scalability

So there is a lot of unsolved points related to data transfer, which is good,
since I can make it relevant to my job. I would say, "disk scarcity" and
"federated availability zones" look like promising areas for FTS.

As it has been suggested, a "Systematic Literature Review" /
"Systematic Mapping Study" will likely help to reduce the scope and identify
a research area.
