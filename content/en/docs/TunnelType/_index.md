---
title: "IPsec Mode & Tunnel Type"
linkTitle: "IPsec Mode & Tunnel Type"
weight: 60
description: >
  IPsec mode & tunnel types supported by MyIKEv2
---

MyIKEv2 support both transport mode and tunnel mode, for tunnel mode, following two types of IPsec tunnel as either tunnel initiator or responder are supported:

* LAN-to-LAN (L2L) tunnel
* Remote-Access (RA) tunnel

The key difference is RA tunnel uses IKEv2 configuration payload while L2L does not.

For client role, the tunnel type is specified by value of "ratunnel" in setup file:

* true: RA
* false: L2L

For gateway role, the tunnel type is based if the peer include configuration payload in IKE_AUTH request.

## LAN-to-LAN Tunnel

L2L tunnel is typically used for router-to-router connection, a.k.a LAN-to-LAN; 
```
LAN_1 --- R1 ----L2L_Tunnel --- R2 --- LAN_2
```
The clear traffic of L2L tunnel could come either from local host or from other hosts behind it; with above example, the clear traffic R1 forward into L2L tunnel could either come from R1 locally or from other hosts on LAN_1;

## Remote-Access Tunnel
RA tunnel is typically used for road-warrior remote access, a.k.a cooperate remote-access VPN;

```
client --- RA_Tunnel --- VPN_GW --- Private_LAN
```

Client will typically request an internal address and DNS server address from VPN_GW to access Private_LAN; the assignment is done via negotiation of IKEv2 configuration payload. 

Client in this case it typically a PC or a mobile device; the all clear traffic client send into RA tunnel comes from local host, and uses assigned internal address as the source address of the clear traffic; the negotiated TSi is typically the assigned_addr/32 or assigned_address/128.

## Transport Mode
To use transport mode, configure `childlist->tunnelmode: false` in setup file, which is a per CHILD_SA configuration. 


## Fastpath
see [Fast Path](../fastpath/)