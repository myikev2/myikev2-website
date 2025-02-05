---
title: "Change Logs"
linkTitle: "Change Logs"
weight: 1000
description: >
  What's new?
---

## ver 5.0 2/3/2025
- reworked CLI, comply to POSIX/GNU-style --flags, now also support shell auto completion; there are some NBC in CLI parameter
- add support RFC8784 
- add compact command to remove default setting in a setup file 


## ver 4.7 1/26/2024

- reworked CLI and YAML file support
- adding HASH_ALG_NOTIFY value output in debug
- fix a bug when reply to CHILD_SA rekey in transport mode, the USE_TRANSPORT notify is missing
- now HASH_ALG notify debug output shows actual ALG name
- now when receiving a IKE packet contains non-ESP marker on conn where NAT is not enabled is supported
- now GW sendIKESAINITResp use the the conn to send response same as where the request is received
- fix dshash related bug
- fix pfs group related bugs
- fix a bug where MyIKEv2 fragment IKE_SA_INTI pkt, which is not suppose to do

## ver 4.6 4/14/2023

- no longer require libpcap or a specific version of glibc
- fix a bug of eap-file
- fix some bugs on fastpath



## ver 4.5 2/27/2022

- add peerid/matchpeerid for client tunnel

## ver 4.42 11/25/2022

- fix a bug that might cause CHILD_SA rekey fail on fastpath in scale test
- add setupinterval configuration for ping


## ver 4.41 11/25/2022

- fix a memory leak bug when using UDP ping in scale test

## ver 4.4 11/22/2022

- add UDP ping
- enhanced client eap-file, and gateway EAP implementations, more stable and performant

## ver 4.3 11/18/2022

- improve performance of netlink based fastpath
- fix a bug cause wrong eapradiusss could cause loading radius pcap to panic
- change default test running time to 10 min


## ver 4.22 10/27/2022

- fix a bug that could cause logging to stop by certain formatted payload 

## ver 4.21 10/17/2022

- change default IKEv2 fragmentation MTU to 1100

## ver 4.2 10/5/2022

- use netlink as most of fastpath API calls, which increase the fastpath setup performance
- fix some bugs related to eapsnoop
- RA client IKE_AUTH request now include CFG_ATTR_INTERNAL_IP4_NETMASK

## ver 4.0 8/30/20222

- Support multiple SA proposals for IKE and CHILD SA
- add TCP encap support RFC8229
- add support INVALID_KE notification for tunnel responder
- change EAP support for gateway to use EAP-start
- add force using UDP encap
- rewrite NAT detection related code
- now client reauth will wait for all IKE_SA deletion finish before start new dial
- many bug fixes


## ver 3.0 03/09/2021

- Add myikev2 daemon & controller support, controller could create tests based on a user specified recipe on multiple machines or multiple name spaces on same machine; see documentation for details;
- Add support for IPsec transport mode
- Add support for RFC4754, IKE and IKEv2 Authentication Using the Elliptic Curve Digital Signature Algorithm (ECDSA)
- Add tunnel flapping, which keep flapping tunnels up and down, could be used for stress testing
- Add crash log file
- Enhance ping test
- varies other enhancements and bug fixes 

## ver 2.0 12/19/2019

- Add tunnel responder support (IPsec gateway); support all existing MyIKEv2 features
- Add ed25519 support 
- Add unique EAP credential per tunnel support for eap-snoop mode
- Add auto generating ping src/dst address based on traffic selector
- refined log support, fixed out-of-order issue
- now allowing omit default config in ikeconf and pingconf
- add jitter for rekey
- add API/interactive-CLI for list and dump CHILD_SA
- fixed some rekey bugs
- Fixed many bugs, more efficient memory usage and better stability in large scale tests

## ver 1.7 7/6/2019

- add gRPC based API support, see doc for detail
- add maxlossrate setting for built-in ping
- add psummary interactive command (show summary of ping results)
- fixed a bug of built-in ping

## ver 1.6, 5/19/2019

- Add MOBIKE (RFC4555) , include fastpath support, see doc for detail
- Add built-in Ping test, see doc for detail
- Add "desc" setting in setup file

## ver 1.52, 4/28/2019

- fixed a bug that cases issues on LAN-to-LAN tunnel's fastpath

## ver 1.51, 4/25/2019

- fixed a bug that causes fastpath stop working after CHILD_SA rekey

## ver 1.5, 4/24/2019

- add IKEv2 fragmentation reassembly part, now the feature is completed
- EAP only auth (RFC5998)
- New crypto:
    * ChaCha20/Poly1305(RFC 7634), for both IKEv2 and fastpath
    * curve25519 (dhgrp 31)
- IKEv2 Repeated Authentication (RFC4478)
- add IKEv2 initial_contact support
- Some bug fixes

## ver 1.4, 4/10/2019

- Now support 100K tunnels on a single socket 10 core Xeon CPU (E5-2650 v3), tested under following setup:
    * HyberThreading enabled
    * MyIKEv2 runs in a centos 7 VM (Qemu/KVM), with following resource allocated to VM:
        * 17 HT cores
        * 80G memory
        * A dual port 10GE NIC (Intel 82599ES) via PCI-Passthrough
    * Key MyIKEv2 Config:
        * auth: psk
        * crypto: aes-cbc-128 and sha1, DH Grp 14, PFS enabled
        * lifetime: IKE 30min, CHILD 15min (MyIKEv2 is the rekey initiator)
        * DPD: enabled, interval 30s
    * Memory Consumption:
        * After all tunnels created, memory Consumption is less than 56G
- Add in-memory logging; enabled via by setting "loginmem" to true in setup file; if enabled, MyIKEv2 will keep logging msg in memory util test finish, it then write logs into file
- Tunnel retry interval and max retry are now configurable via "tunnelretryinterval" and "tunnelmaxretry" in setup file; this is the time MyIKEv2 wait before retry if previous tunnel creation failed
- IKE request re-tranmission interval and max retry are now configurable via "retraninterval" and "maxretran" in setup file; 
- Test result output has been enhanced, setup rate has been added, with option to use JSON format with "-j" command parameter
- adding first phase support of IKEv2 fragment (RFC7383), only fragmentation is supported in this release