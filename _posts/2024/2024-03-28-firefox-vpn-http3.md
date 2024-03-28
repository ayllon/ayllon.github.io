---
title: Firefox and FortiClient, connectivity problems related to http3
---

I am not going to pretend I know exactly what is happening. However,
when my company introduced FortiClient, I had trouble connecting to some
sites, especially some from Google, such as Calendar.

Opening Google Calendar would be as slow as molasses. Things would get better,
perhaps, after one or two failed refreshes. Once the connection was established,
it kept working mostly OK.

But Feedly was unusable. I could not log in. Every time I clicked on "Log in with Google",
the site became unresponsive, and a bunch of `NS_BINDING_ABORTED` would show up on
Firefox's Network Monitor.

Eventually, I gave up and switched to Chrome, which didn't show this behavior.

But! Someone else from the company mentioned having trouble with FortiClient's
[MTU](https://en.wikipedia.org/wiki/Maximum_transmission_unit) of 1200 and Docker.
It seems that 1200 may not be enough for QUIC either (used by HTTP/3). Hence, the 
connection failures, and the completely miserable experience with Firefox.

(Also, [it seems Chromium-based browsers are better able to fallback to HTTP/2](https://gist.github.com/jj1bdx/1adac3e305d0fb6dee90dd5b909513ed) when this happens.)

So, in Firefox, I went to the address bar, typed `about:config`, looked for
`network.http.http3.enable`, and disabled it. This fixed my issues with the bad 
connectivity, and I could go back to Firefox again 🥳.
