---
title: "Data Path"
linkTitle: "Data Path"
weight: 70
description: >
  MyIKEv2 datapath
---

The datapath of MyIKEv2 rely on Linux kernel XFRM Framework, it uses "ip xfrm" and "ip route" command to add XFRM state/policy and corresponding routes; 

Datapath is installed when "installfastpath" in setup file is set to "true"; if only IKEv2 protocol tests are required, this setting could be turned off to save resources;

MyIKEv2 use route table 330 to store all generated routes;

For TCP encap, a linux kernel >= v5.6 is required.

## Datapath of Remote-Access Tunnel

Remote-Access Tunnel is where IKEv2 configuration payload is used to request IPv4/IPv6 Internal address/DNS from DUT; typical use case is road-warrior;

Following is an example of installed XFRM state/policy and route:

The example tunnel has following attributes:

* local tunnel address is 11.1.0.2, peer's is 11.1.0.1
* negotiated TSi is 192.168.1.100/32 (the assigned internal address)
* negotiated TSr is 192.168.2.0/24
* enp0s10 is the egress interface to reach peer

```
root@myikev2:~# ip xfrm state
src 11.1.0.1 dst 11.1.0.2
        proto esp spi 0x73ff0a18 reqid 1 mode tunnel
        replay-window 0
        auth-trunc hmac(sha1) 0xf28ac369864e7d46421651fb49e753a314e40b91 96
        enc cbc(aes) 0x3f21e63bdab65797b5607a4f1f4137ee
        anti-replay context: seq 0x0, oseq 0x0, bitmap 0x00000000
        sel src 0.0.0.0/0 dst 0.0.0.0/0
src 11.1.0.2 dst 11.1.0.1
        proto esp spi 0xc1cac31d reqid 1 mode tunnel
        replay-window 0
        auth-trunc hmac(sha1) 0x26e8f81adb58014aed8893fa37ec2b951768edbe 96
        enc cbc(aes) 0x4feb7f5e23b1a7afe2960db37d833b3e
        anti-replay context: seq 0x0, oseq 0x0, bitmap 0x00000000
        sel src 0.0.0.0/0 dst 0.0.0.0/0
root@myikev2:~# ip xfrm policy
src 192.168.2.0/24 dst 192.168.1.100/32
        dir fwd priority 0
        tmpl src 11.1.0.1 dst 11.1.0.2
                proto esp reqid 1 mode tunnel
src 192.168.2.0/24 dst 192.168.1.100/32
        dir in priority 0
        tmpl src 11.1.0.1 dst 11.1.0.2
                proto esp reqid 1 mode tunnel
src 192.168.1.100/32 dst 192.168.2.0/24
        dir out priority 0
        tmpl src 11.1.0.2 dst 11.1.0.1
                proto esp spi 0xc1cac31d reqid 1 mode tunnel
root@myikev2:~# ip route list table 330
192.168.2.0/24 dev enp0s10 proto static scope link src 192.168.1.100
```
The assigned internal address 192.168.1.100 is used as source address for traffic destined to 192.168.2.0/24.

## Datapath for LAN-to-LAN Tunnel

LAN-to-LAN is where IKEv2 configuration payload is not used; typical use case is to connect two routers;

Following is an example of installed XFRM state/policy and route:

The example tunnel has following attributes:

* local tunnel address is 2001:dead::1, peer's is 2001:dead::ffff
* negotiated TSi is 2001:aaaa::1/128 
* negotiated TSr is 2001:abcd::1/128
* enp0s10 is the egress interface to reach peer

```
root@myikev2:~# ip xfrm state
src 2001:dead::ffff dst 2001:dead::1
        proto esp spi 0x3be2f2a6 reqid 1 mode tunnel
        replay-window 0
        auth-trunc hmac(sha1) 0x9a402be9b41c0c28511b1541ae914b9446e8f2b4 96
        enc cbc(aes) 0x0ac7738a8d0c72a083cef19eab042609
        anti-replay context: seq 0x0, oseq 0x0, bitmap 0x00000000
        sel src ::/0 dst ::/0
src 2001:dead::1 dst 2001:dead::ffff
        proto esp spi 0xc6dcb76d reqid 1 mode tunnel
        replay-window 0
        auth-trunc hmac(sha1) 0x2c408618a696a043465fe218a4404785aeb61a58 96
        enc cbc(aes) 0xb0c44374d7e5a903fc78a92a691b06cf
        anti-replay context: seq 0x0, oseq 0x0, bitmap 0x00000000
        sel src ::/0 dst ::/0
root@myikev2:~# ip xfrm policy
src 2001:abcd::1/128 dst 2001:aaaa::1/128
        dir fwd priority 0
        tmpl src 2001:dead::ffff dst 2001:dead::1
                proto esp reqid 1 mode tunnel
src 2001:abcd::1/128 dst 2001:aaaa::1/128
        dir in priority 0
        tmpl src 2001:dead::ffff dst 2001:dead::1
                proto esp reqid 1 mode tunnel
src 2001:aaaa::1/128 dst 2001:abcd::1/128
        dir out priority 0
        tmpl src 2001:dead::1 dst 2001:dead::ffff
                proto esp spi 0xc6dcb76d reqid 1 mode tunnel
root@myikev2:~# ip -6 route list table 330
2001:abcd::1 dev enp0s10 metric 1024 pref medium
```

In case of LAN-to-LAN tunnel, for each TS in TSr, MyIKEv2 will create a route with the smallest prefix length that covers the address range in the TS;