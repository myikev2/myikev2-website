---
title: "MOBIKE (RFC4555)"
linkTitle: "MOBIKE (RFC4555)"
weight: 100
description: >
  MyIKEv2 MOBIKE (RFC4555) implementations.
--- 

MyIKEv2 supports MOBIKE (IKEv2 Mobility and Multihoming Protocol, RFC4555) as either tunnel initiator or tunnel responder;
The support includes:

* change own tunnel address
* change peer's tunnel address
* change both own and peer's tunnel address
* Accept peer updates of its tunnel address
* Fastpath support

## Configuration 

client role:
There are following settings in setup file for MOBIKE

* mobike: set to true to enable MOBIKE
* mobikeaddrpertunnel: the number of own tunnel address for each tunnel
* mobikeiplifetime: the amount of time MyIKEv2 wait before change to next address
* mobikechangeaddrtype: own-only|peer-only|both; own-only only change own address, peer-only only changes peer address; both changes both own and peer address
* ikesa->disallownat: set to true to include NO_NATS_ALLOWED notification in IKE_AUTH request and UPDATE_SA_ADDRESSES request.

gateway role:
There are following settings in setup file for MOBIKE

* mobike: set to true to enable MOBIKE
* mobikeaddrpergw: the number of own tunnel address for the gateway


## How does it work (client role)?
Once enabled, MyIKEv2 will change its own address and/or peer's address (based on mobikechangeaddrtype) every mobikeiplifetime; 
The mobikeaddrpertunnel specifies how many own address each tunnel has, the addresses are allocated for each tunnel as following:

* for example, if startclntaddr is "192.168.1.1/24", and mobikeaddrpertunnel is 3, then 1st tunnel get 3 addresses: 192.168.1.1, 192.168.1.2, 192.168.1.3; 2nd tunnel get 3 addresses: 192.168.1.4,192.168.1.5,192.168.1.6; and so on ...

The available peer's tunnel addresses are peeraddr in setup file plus the addresses signed by peer via ADDITIONAL_IP4_ADDRESS and ADDITIONAL_IP6_ADDRESS Notify Payloads, in IKE_AUTH response and peer initiated information request.



## How does MyIKEv2 pick next address (client role)?
This depends on mobikechangeaddrtype:

* own-only: next address in own address list, which is specified by startclntaddr and mobikeaddrpertunnel
* peer-only: next address in the latest available peer address list, which is the current in-using peer address plus address signed by peer 
* both: the next combination of own address list and latest peer address list


## How does it work (gateway role)?
MOBIKE address change is driven by tunnel initiator, so as tunnel responder, MyIKEv2 only respond to peer's address change request.


## Linux Reverse Path Filter 
Reverse path filtering is a mechanism supported by the Linux kernel to check whether a receiving packet comes in via right interface. the purpose is to prevent address spoofing used in DoS attack;

However with MOBIKE, in certain test setup, with address change, the IPsec packet might come in via a different interface, which will fail reverse path checking, and depends on kernel setting, the packets might get dropped;

So to simplify the test setup, user could choose to disable reverse path filter in Linux as following:

### Disable Linux IPv4 Reverse Path Filtering
```
sysctl -w net.ipv4.conf.<inteface1-name>.rp_filter=0
sysctl -w net.ipv4.conf.<inteface2-name>.rp_filter=0
sysctl -w net.ipv4.conf.all.rp_filter=0
```

### Disable Linux IPv6 Reverse Path Filtering

```
ip6tables  -t raw -A PREROUTING  -m rpfilter -j ACCEPT
ip6tables -t raw -A PREROUTING  -m rpfilter --invert -j ACCEPT
```