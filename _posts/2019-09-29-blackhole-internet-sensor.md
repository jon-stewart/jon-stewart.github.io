---
title:  "Blackhole - Internet Sensor"
tags:
  - honeypot
  - sensor
  - TPROXY
  - iptables
related: false
---

How do you capture previously unseen attack traffic?

The typical honeypot is designed for a specific port (TCP/UDP) and application layer protocol with acceptable responses designed to elicit further interaction known ahead of time.  This, of course, falls short in catching those emerging threats.

Enter Linux IPtables TPROXY.  This magical module enables one to listen on all ports TCP and UDP, capture the source IP address, target port, and data.  This work was largly inspired by a CloudFlare blog article [Abusing Linux's firewall](https://blog.cloudflare.com/how-we-built-spectrum/).  A worthwhile read as it goes into some depth on the how and why.

Sample code using TPROXY can be found at [github](https://github.com/jon-stewart/blackhole). Please drop the privileges from root after configuring the ports. 

Hook the sensor up to Apache Kafka and you now have a pipeline for the deluge of internet noise that you are about to receive.  The real trick is being able to filter out interesting and relevant data.  An appropriate analogy is panning for gold in sewage.

Some a la [GreyNoise](https://greynoise.io) view this data as valuable in so far as it is worthless, 'anti-intelligence' that can be best used as a filter.  Whether this applies to honeypots in general or just sensors depends on how long one has been working with them.
