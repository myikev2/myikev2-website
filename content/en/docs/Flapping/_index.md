---
title: "Tunnel Flapping Test"
linkTitle: "Tunnel Flapping Test"
weight: 120
description: >
---

Tunnel flapping is a IKEv2 stress testing feature allows user to specify a number of **client** tunnels doing following step:

1. Tunnel is created for the 1st time
1. wait a interval
1. remove the tunnel and re-establish it, Goto Step-2


This feature is configured by `flapconf` section in setup file:
```
flapconf:
  # enable/disable tunnel flapping
  flapping: false
  # number of tunnel flapping, must <= numberoftunnels
  # -1 means same as numberoftunnels
  numoftunnel: -1
  # the interval between two dials is a random number between minflapinterval and maxflapinterval
  # minflapinterval must >= 10s
  minflapinterval: 30s
  maxflapinterval: 1m0s

```
