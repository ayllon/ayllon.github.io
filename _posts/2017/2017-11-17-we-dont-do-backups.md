---
title: We don't do backups
---

At the beginning, we used to backup the FTS database daily, because, well,
we all know backups are good, aren't they?

You probably have also heard that you better test your backups, or you
may run [into big problems](https://techcrunch.com/2017/02/01/gitlab-suffers-major-backup-failure-after-data-deletion-incident/)
if - when - you need to recover from them.

Well, we didn't test recovering from backups either, but that's not exactly
the point of this entry.

The story started just the day before the 2015 Christmas closure at CERN.
The [FTS](http://fts3-service.web.cern.ch/) service started to slow down,
until it completely froze. No transfer was running, nothing was happening.
Just the day before leaving for two weeks of holidays. Great, just great.

Luckily it happened to be easy to debug: the backup procedure would try
to acquire a global read-lock, take the snapshot, and then release. However,
a long query happened to be running at the same time - this query has, since then,
improved a lot - so the lock would block waiting for the long query, but
blocking a subset of all the tables already, stopping any update that was trying
to happen.

The symptom, when doing a `SHOW PROCESSLIST`:

```sql
FLUSH TABLES WITH READ LOCK;
```

Plus one, or a few, very long queries, and a bunch of blocked ones.

You may ask, "why would a `SELECT` block a read-lock?". Well, in this case,
[it does](https://www.percona.com/blog/2012/03/23/how-flush-tables-with-read-lock-works-with-innodb-tables/).

With barely time to think it twice, we had to kill the backup for the moment,
recover the service, and with no time to do a patch release, decide what
to do next.

And we decided to *disable the backups for the two weeks*. Gasp!

And then we came back from holidays, and decided never to enable them again.
Double gasp!

Why? Well, in our case, if we do a backup at 12.00, and then we have a
catastrophic failure at 14.00 and lose all the data, what should we do?

### 1. Recover from the backup
What happens then? We go, load the backup, start the service... and start
running transfers that already run two hours ago. If we are lucky, they
succeed. If we are not, they fail. And, if they fail, the destination file will
no longer be there, while the experiment's framework think it is, because we
said we transferred the file two hours ago, and they kept moving on.

So now, trying to recover from our data loss, we are losing **user data**.
And that's absolutely a no-go.

### 2. Do not recover, and start with an empty queue
Risking losing user data is unforgivable. And, besides, the framework on which
the experiments rely, are capable of resubmitting a lost job.
Therefore, we do not backup the queue anymore.

We do backup the configuration, though!

### Moral of the story
Backups are part of your system as much as any other. Before even starting
to test recovering from backup, you need to understand how losing the data,
and then recovering, affects your use case.

And then, yes, test your backups.
