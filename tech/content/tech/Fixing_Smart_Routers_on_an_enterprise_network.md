+++
date = "2017-02-10T19:48:31-05:00"
title = "Fixing *Smart* Routers on an enterprise network"
draft = true

+++

# Fixing "Smart" Routers on an enterprise network

In the house I live in, we have a pretty complex network. By complex, I mean it's a standard corporate network, without the money to properly set up or maintain it, as well as a new network admin every 2 years or so, with partially working patches when things go wrong.

The network involves a cisco router, 4 switches, 9 UniFi APs, and several personal routers of different brands since the building we live in is not conducive to wifi.

When someone plugs in their own router, it's the job of the admin to turn of the DHCP server, since there can only be one on a network without breaking the network, unless the network is partitioned correctly. That being said, sometimes people ignore you and just plug stuff in anyway, which tends to break things. The internet is pretty slow despite our house having 1 Gb service, so naturally the blame falls to either the cisco equipment or the APs, but it's very hard to tell.

When I was tasked with 'fixing the wifi', there were a few things that had to be done first. Document our set up, which had no documentation, figure out the health of the network, find the points of failure, then fix those points of failure. 

& Note: Linksys smart wifi is stupid, doesn't allow basic functionality, found hidden tabs in the HTML, had to navigate to pages manually (advanced-wireless.html).