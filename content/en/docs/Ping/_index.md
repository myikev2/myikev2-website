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
  ptype: icmp
  udpport: 9922
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
* ptype: type of ping, icmp or udp; refer to below section for details of UDP
* udpport: UDP port in case of UDP ping, used as both src and dst
* destaddr: the ping destination for 1st tunnel
* deststep: the number of step increase for destination addr of each following tunnel; for example with above config, 2nd tunnel's ping destination is 192.168.1.101, 3rd tunnel is 192.168.1.102 ...
* srcaddr: specifies the ping source address for 1st tunnel; if empty, it means it let OS automatically select source; leave empty for RA tunnel
* srcstep: the number of step increase for src addr of each following tunnel; only applies when srcaddr is not empty
* interval: interval between send ping ECHO request
* setupinterval: interval between creating two consecutive ping tasks; shouldn't be too small in scale test to avoid all ping task sending at the same time.
* pktlen: the size of ping packet send; note: the actual IP pkt size is bigger than this, since this only specifies ping payload size.
* maxlossrate: the max allowed packet loss rate in percentage, a float number between 0-100; if the packet loss rate exceed this value, then an error event will be generated;
* holdtime: the amount of time system waits before start ping, after all tunnel are created (as client role) or gateway is created (as gateway role)

## UDP Ping
To use UDP ping, myikev2 echo server need to be running as ping target, which will reflect received UDP packet back to the sender.

```
  myikev2_udp_ping <---------> myikev2_echo_svr
```
echo server could be started vi command `myikev2 echosvr`.
```
echosvr: start UDP echo svr
  -count uint
        number of listening IP (default 1)
  -port int
        listening port (default 9922)
  -startip string
        starting listening IP
  -step uint
        step (default 1)
```
the echo server listening one or multiple addresses with the specified port, the step specifies delta between two consecutive addresses, for example command `myikev2 echosvr -startip 1.1.1.1 -count 3 -port 3344 -step 2` will create a server listening on 3 addresses 1.1.1.1, 1.1.1.3 and 1.1.1.5 with listening port 3344.

note: echo server will automatically add listening addresses to interface lo, so user doesn't need to add them manually, it also means using this command requires root privilege. 



## CLI
There are following shell CLI commands relate to ping test:

* MyIKEv2 CLI:
  * `psummary`: display the ping stats
  * `clearping`: clear ping stats

* Controller CLI:
  *`list`: list instance status, which include ping stats
  *`clearping`: clear ping stats

## Limitation
The built-in ping is currently not designed for big load testing, which could cause inaccurate result, specially icmp ping.