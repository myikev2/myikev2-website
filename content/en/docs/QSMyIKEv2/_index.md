---
title: "Quantum Safe MyIKEv2"
linkTitle: "Quantum Safe MyIKEv2"
weight: 110
description: >
---

MyIKEv2 supports following quantum safe features:

- RFC8784: Mixing Preshared Keys in the Internet Key Exchange Protocol Version 2 (IKEv2) for Post-quantum Security 


## RFC8784
MyIKEv2 supports rfc8784 for both client and gateway role:

- client: the feature is enabled when ppk.id in setup is not empty 
- gateway: the feature is enabled when ppk.id in setup is not empty and received USE_PPK notification in IKE_SA_INIT request

if ppk.ppkrequired is true, then IKEv2 negotiation will fail if PPK negotiation fails 