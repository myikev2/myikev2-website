
---
title: "Documentation"
linkTitle: "Documentation"
weight: 10
menu:
  main:
    weight: 10
---


MyIKEv2 is an IKEv2/IPsec testing tool for Linux.

It supports following features:

- Testing Focus:
    - Simple setup: single executable with single setup file
    - Orchestrated setup: multiple instances on one or multiple servers, orchestrated by a central controller
    - gRPC based API support for test automation
    - Capable of creating large number of concurrent IPsec tunnels; **100k** tunnels tested on single 10 core Intel Xeon CPU; 
- Quantum safe: RFC8784
- Full IPv4/IPv6 combinations support
- Both Tunnel and Transport mode support
- TCP Encapsulation support (RFC8229)
- IKEv2 implementation based on RFC7296    
    - Initiator and responder
    - Multiple transform support
    - IKE SA rekey, initiator and responder
    - Child SA rekey, initiator and responder
    - PFS
    - Cookie
    - Configuration payload
    - Traffic Selector
    - NAT Traversal
    - Repeated Authentication (RFC4478)
    - IKEv2 Fragmentation (RFC7383)
    - MOBIKE (RFC4555)
- Authentication:      
    - PSK
    - Certificate                
      - key type: RSA/ECDSA/Ed25519
      - IKEv2 digital signature, RFC7427 
      - IKEv2 ECDSA authentication, RFC4754 
    - EAP
- Crypto:
    - Encryption: AES-CBC/AES-GCM_12/AES-GCM_16/Chaha20-Poly1305
    - Integrity: MD5/SHA1/SHA256/SHA384/SHA512
    - DH Grp: 1/2/5/14/15/16/17/18/19/20/21/31    
- Built-in Ping Test
- Fastpath: linux kernel xfrm

