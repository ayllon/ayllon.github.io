---
title: OpenVPN @ Home
---

For a couple of years now we have had at home a Raspberry laying around,
never quite finding its true purpose in life.

Finally, this month I decided to give it a use, and run an OpenVPN inside my
home network.

## Why?
I recently installed a NAS at home. As a lot of people these years, most of
my memories are on digital format (photos, videos,...) and I am not super keen
on losing them. Also, they are starting to account for quite some GB...

I want to be able to access the NAS when I am away, but I am reluctant to
open it to the outside world, for the network is dark and full of terrors.

Installing a VPN, on the other hand, allows me to connect to the NAS as if I
were on my home network because, well, *I am*.

It implies opening the VPN port to the outside, but the attack surface is considerably
reduced, and protected by public/private key cryptography. This also means that
when I connect to my VPN I know I am actually connecting to it and
not something else. Same technology as the 's' in 'https'.

As an extra, this will also allow me to encrypt my traffic when connected
to public networks.

## How
There isn't much to say about this, really. I am not going just to reproduce
one of the many guides that there are out there. Overall:

* Install and update Raspbian
* Follow the guide at [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-debian-8),
which is pretty well detailed
* Configure a Dynamic DNS
* Open on the NAT the port for the OpenVPN, which by the way I have changed to
a non standard one

Being a bit paranoid, I am considering to run also [Tripwire](https://en.wikipedia.org/wiki/Open_Source_Tripwire) on the Raspberry,
*si jamais*, but haven't set it up yet.
