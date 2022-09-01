---
title: "MyIKEv2 License"
linkTitle: "MyIKEv2 License"
weight: 140
description: >
---

MyIKEv2 require a valid license file, without it, it will run in trial mode, which has following limitations:

* max number of tunnels is limited to 10
* max running time is limited to 30 minutes 

## Provision License File
By default, MyIKEv2 expects license file to be as /etc/myikev2.lic; however this could be overridden by ```-l``` parameter as ```myikev2 exec -f <setupfile> -l <licensefile>```