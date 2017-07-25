---
title: Let the users do the testing for you
---

One problem we have when we want to test FTS3 is to do a reasonable approximation
to a production workload, and to what's happening (or may).

This is hard for mainly four reasons:

1. Experiments work-flows change a lot. If we try to program a tool to generate load,
it will be outdated in months.

2. They transfer between O(200) different storages, with different software,
different versions, different resources... We do not have access to most of them.

3. To those we do, the allocated resources vary wildly, and those machines
that can be used by `dteam` (the virtual organization to which developers belong)
tend to be fragile and break often. This causes us to spend a non negligible amount
of work to debug issues that are *not* on our side, and that we can't control.

4. Last, but not least, there are some error scenarios we may want to test, but
that should not happen in production, but they do happen. For instance, think
of a SIGSEGV or SIGABRT triggered from a transfer. You would expect the underlying
libraries *not* to do that, but they will, and we better cover the possibility.
Also, connection severed by the peer, TLS handshake errors. Those are tricky to
trigger in a controlled manner.

## Interlude: Mock plugin
In respecto to the last item. FTS3 uses a library, called GFAL2, that abstracts
away different protocols. It is plugin based, so if you want to add a new
protocol, you "just" need to write a new plugin.

GFAL2 is already tested with a set of different storages behind, although it
hardly can cover a SIGSEGV from the, say, Globus libraries to which it links.

From FTS3 perspective, we do not care so much then about real interaction, but
rather possible outcomes from these interactions, regardless of the actual protocol.

So to cover these sort of potential malfunctions, we wrote a "mock plugin", which
implements a fake protocol (`mock://`), and we can influence the plugin behavior
based on query parameters.

For instance, we can write a url like `mock://host/path?signal=11`, and this will
trigger the plugin to do a `raise(11)`, which from the point of view of FTS is
as good (or bad) as an actual segmentation fault.

So, thanks to this plugin, we can test if FTS3 can recover from a transfer
killed by SIGSEGV, SIGABRT or even a SIGKILL.

It does.

## Doppelganger

> A doppelgänger or doppelga(e)nger (/ˈdɒpəlˌɡɛŋər/ or /-ˌɡæŋər/;
> German: [ˈdɔpl̩ˌɡɛŋɐ], literally "double-goer")
> is a look-alike or double of a living person, sometimes portrayed as a paranormal phenomenon,
> [[1]](https://en.wikipedia.org/wiki/Doppelg%C3%A4nger)

For the other three points, we use a custom tool called "Doppelganger".
Since reproducing the production workload is hard, we don't. This tool
subscribes to the production message broker, and listen to messages that notify
of new and completed transfers.

Using the completion messages, it keeps some statistics about the percentage of
errors, and from those, which errors are being triggered. It also stores
the average throughput.

On the other side, every time it sees a submission message, it goes to this
statistics table, picks the throughput, and a error with the same probability as
seen in production, replaces the actual protocol with `mock://`, and injects
the error into the query arguments if necessary.

![Doppelganger]({{baseurl}}/img/doppelganger.png)

For instance, let's say that we have these statistics for the link
`gsiftp://source` => `gsiftp://destination`

| Throughput | Success Rate | Errors |
| ---------- | ------------ | ------ |
| 100 | 0.8 | ENOENT (0.4), ETIMEDOUT (0.6) |

Then, Doppelganger receives a transfer of 200MB from `gsiftp://source/path/file` to `gsiftp://destination/path/file`.
It will go to the table and throw a dice. With a 0.2 chance the replayed submission
will be an error. Let's assume it gets an error, and it will be ETIMEDOUT.
It will then resubmit to the development instance as

`mock://source/path/file` => `mock://destination/path/file?transfer_errno=110&time=2`

Someone may ask why not to replay the submission when we receive the final state.
The reason is that FTS3 throttles the number of active transfers that run between
two storages, so then we wouldn't be submitting at the rate production is receiving,
but at the rate they are finishing.

### "Real mode"
Additionally, Doppelganger is also capable of replaying the submission but using
real storages. In this case we can not inject errors, but we can get an actual
interaction with the storage.

In this mode, when a transfer is received, the tools checks the storage
implementation of each side of the transfer, picks one with the same software
from a curated list of servers we know we can access and rely on, and resubmit
a similar transfer using pre-populated files.

## Pre-production
Last, but not least, to test also how the experiments frameworks integrate with
the service, they run a subset of production transfers on the pre-production
instance. This allows for a complete end to end coverage, with a reasonable
approximation to the "actual" usage.
