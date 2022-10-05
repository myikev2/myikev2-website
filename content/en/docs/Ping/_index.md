---
title: "Built-in Ping test"
linkTitle: "Built-in Ping test"
weight: 110
description: >
---

MyIKEv2 has a built-in ping test feature; MyIKEv2 could automatically start ping sessions after tunnels are created, the ping session will keep running until test finishes, then reports number of packets sent/received.
The number of ping sessions is determined by:

* client role: the number of successfully created IPsec tunnels
* gateway role: numberoftunnels in setup file

## Config
Ping test is enabled by setting ```destaddr``` in ```pingconf``` section of setup file:
```
pingconf:
  autoaddr: false
  destaddr: "192.168.1.100"
  deststep: 1
  srcaddr: ""
  srcstep: 1
  interval: 1s
  pktlen: 64
  maxlossrate: 10
  holdtime: 10s
```

* autoaddr: if set as true, then src/dst address of ICMP ECHO request will be the first address in TSi/TSr address range
* destaddr: the ping destination for 1st tunnel
* deststep: the number of step increase for destination addr of each following tunnel; for example with above config, 2nd tunnel's ping destination is 192.168.1.101, 3rd tunnel is 192.168.1.102 ...
* srcaddr: specifies the ping source address for 1st tunnel; if empty, it means it let OS automatically select source; leave empty for RA tunnel
* srcstep: the number of step increase for src addr of each following tunnel; only applies when srcaddr is not empty
* interval: interval between send ping ECHO request
* pktlen: the size of ping packet send; note: the actual IP pkt size is bigger than this, since this only specifies ping payload size.
* maxlossrate: the max allowed packet loss rate in percentage, a float number between 0-100; if the packet loss rate exceed this value, then an error event will be generated;
* holdtime: the amount of time system waits before start ping, after all tunnel are created (as client role) or gateway is created (as gateway role)

## CLI
There are following CLI commands relate to ping test:

* MyIKEv2 CLI:
  * `psummary`: display the ping stats
  * `clearping`: clear ping stats

* Controller CLI:
  *`list`: list instance status, which include ping stats
  *`clearping`: clear ping stats

## Limitation
The built-in ping is currently not designed for big load testing, which could cause inaccurate result.