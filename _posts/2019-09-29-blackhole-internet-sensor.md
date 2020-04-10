---
title:  "Blackhole - Internet Sensor"
tags:
  - honeypot
  - sensor
  - TPROXY
  - iptables
related: false
---

On capturing internet noise

Honeypot tend to have a common shortcoming - they only capture attacks against known vulnerabilities on a specific port, L4 protocol, and application.  What if it was possible to capture and identify new and interesting traffic on any port?  Could it catch zero days and new exploits?

Enter IPtables TPROXY.  This magical module allows a program to listen on all ports and supported protocols, capture the source IP address, target port, and data sent.  This work was largly inspired by a CloudFlare blog article [Abusing Linux's firewall](https://blog.cloudflare.com/how-we-built-spectrum/).  A worthwhile read as it goes into some depth on the how and why.

Sample code using TPROXY can be found at [github](https://github.com/jon-stewart/blackhole). Please drop the privileges from root after configuring the ports. 

Hook the sensor up to Apache Kafka and you now have a pipeline for the deluge of internet noise that you are about to receive.  The real trick is being able to filter out interesting and relevant data.  An appropriate analogy is panning for gold in sewage.

Some a la [GreyNoise](https://greynoise.io) view this (and general honeypot) data as valuable in so far as it is worthless, 'anti-intelligence' that is best used as a filter.
