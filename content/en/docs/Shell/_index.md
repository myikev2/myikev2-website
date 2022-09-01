---
title: "MyIKEv2 Interactive Shell"
linkTitle: "MyIKEv2 Interactive Shell"
weight: 40
description: >
  MyIKEv2 Interactive Shell
---


MyIKEv2 provides an interactive CLI when using ```-i``` parameter for ```myikev2 exec``` command. 
It provides following commands:
```
- log
set log level and filter keyword; log [-kw <keyword>] {-l <level>|no}
'log -l no' to disable logging

- uptime
display uptime

- quit
Exit

- psummary
Ping tasks summary
psummary [-start <start>] [-len <len>]

- clearping
clear ping stats

- dump
dump a IKE SA: dump <IKE_SA_OWN_SPI>

- list
show list of IKE SA

- listchild
list all CHILD_SA of the specified IKE_SA: listchild <ike_spi>

- dumpchild
dump specified CHILD_SA: dumpchild <child_spi>

- rekeychild
rekey specified CHILD_SA: rekeychild <child_spi>

- pool
Address pool usage

- shell
go into system shell

- stop
stop all tunnels gracefully

```
## summary

Command ```summary``` prints out current running summary of MyIKEv2:

```
myikev2>>summary
Test setup file: psk.setup
Test started at Wed, 10 Apr 2019 16:01:30 PDT
Tunnel creation started at Wed, 10 Apr 2019 16:01:34 PDT
Tunnel creation finished at Wed, 10 Apr 2019 16:01:51 PDT;
100 tunnel created; took 16.588848566s; avg 6.0281463 tunnels per second
Total number of configed tunnels: 100
IKE SA stats: 100 total, Live 100, 100 has child
Initial: 0              Created: 0
Established: 100        Rekeying: 0
Rekeyed: 0              Closing: 0
Closed: 0
```

## psummary
Command psummary prints out current running summary of ping tasks:

```
myikev2>>psummary
Ping tasks summary:
Total tasks: 10
Total sent pkt:310                      Total recvd pkt:310
```

## log
Command ```log -l <new_level>``` change current STDOUT log level (note: this is different from file log level specified in setup file)

use ```log -l no``` to disable STDOUT logging

optional ```-kw <keyword>``` specifies the keyword to filter log message

use ```log``` without any parameter to display current setting
"

```
myikev2>>log -l 5 -kw 92EA3066263FCF8D
new log level is now 5
now filter log that contains '92EA3066263FCF8D'
```

## uptime
Command ```uptime``` prints out running time:
```
myikev2>>uptime
Current Mon, 21 Jan 2019 16:07:03 PST, Setup started at Mon, 21 Jan 2019 16:03:14 PST,  running time: 3m49s
```

## list 
Command ```list``` print a list SPI of existing IKE SA, it accepts two parameters:
* ```-start <index>```: the index of first returned IKE SA
* ```-len <number>```: the number of returned IKE SA
```
myikev2>>list -len 5 -start 0
following are the IKESA SPIs from 0 to 5
FBD386BA6A47029A
06AE54F413623931
B893DBAE20FF1EF7
D76A0444DE3D89B4
09FC098CE8455955
DE0575F7B4A0DFC9
```

## dump
Command ```dump``` prints the details of a given IKE SA by its SPI:

```
myikev2>>dump 5DFA8EF4CD3FE5F8

Own spi: 5DFA8EF4CD3FE5F8               Peer spi: 5DFA8EF5C308A7AC
Own addr: 11.1.0.2                      Peer addr: 11.1.0.1
State: established                      Close code: n/a
Enc Alg: aes-cbc                        Enc Keylen: 16
Integrity Alg: sha256                   Prf Alg: sha256
Own Auth: psk                           Peer Auth: psk
My Id Type: IPV4_ADDR                   Hash Alg for DS: sha1-96
Use RSA PSS: false                      Psk: pre-shared-key
Initiae DPD: true                       Force DPD: false
DPD interval: 30s                       IKE lifetime: 10m0s
Rekey Margin time: 1m0s                 Jitter: true
Install Fastpath: true                  Keep CHILD_SA history: false
Keep IKE_SA history: false              Enable NAT-T: false
NAT-T keepalive interval: 30s
creation time: Wed, 18 Dec 2019 12:41:26 PST
Last Rcv Pkt time: Wed, 18 Dec 2019 12:41:26 PST
Last send DPD req time: Wed, 18 Dec 2019 12:41:24 PST
Keys:
SK_ei: CB07C6B318ABD9631D9DD2FB9942C3A3
SK_er: C1719A1969EC844EB80F89BF72713C1F
SK_ai: B6D3BE0C7B7789E0A256538C572F34F5CFD9255982D3DA629F79DE3966653C41
SK_ar: BF3C54610BB7D058ABD65559EEDA78C65E361267A316D2522A2277784FD79663
```

## listchild

Command `listchild <IKE_SA_SPI>` return a all CHILD_SA's SPI of the specified IKE_SA;

```
myikev2>>listchild 5DFA9BD4825D0DED
624B2A
```

## dumpchild

Command `dumpchild <CHILD_SA_sPI>` prints details of the specified CHILD_SA;

```
myikev2>>dumpchild 23137B18

State:established
Tunnel mode: true                       Parent IKESA: 5DFBC6027BDA3761
Own spi: 23137B18                       Peer spi: E8B3F2C9
Own addr: 11.1.0.2                      Peer addr: 11.1.0.1
Enc alg: aes-cbc                        keylen: 128
Integrity alg: sha256                   lifetime: seconds:300
ESN: false                              Replay Window Size: 256
creation time: Thu, 19 Dec 2019 10:48:36 PST
OwnTS:
type:ipv4, protocol:any, addr-range: 192.168.100.1 - 192.168.100.1, port-range: 0 - 65535
PeerTS:
type:ipv4, protocol:any, addr-range: 9.9.9.9 - 9.9.9.9, port-range: 0 - 65535
SK_ei:1D1E3F73972DB25C5DE447478D4BB78F
SK_er:0DEF882A30D456B43EE3AE9658ACE4AA
SK_ai:DB85C1E113C1F646F621D56D2504E94FD35ED4180BE82B4DD4E34134C551A570
SK_ar:1C05234E5ACA7F43A01FCEEE05B18B2C3EF2816D22A6CCCFD70CD22B69B5C528
```

## rekeychild
`rekeychild <child_spi>` triggers the rekey for the specified the CHILD_SA 

## psummary 

This command prints out the stats of specified ping tasks:
```
client-1>>psummary -len 10
Ping tasks summary:
8.8.8.1 <-> 9.9.9.1: send 682925 recv 682925, miss 0
8.8.8.2 <-> 9.9.9.2: send 682925 recv 682925, miss 0
8.8.8.3 <-> 9.9.9.3: send 682925 recv 682924, miss 1
8.8.8.4 <-> 9.9.9.4: send 682925 recv 682923, miss 2
8.8.8.5 <-> 9.9.9.5: send 682925 recv 682922, miss 3
8.8.8.6 <-> 9.9.9.6: send 682925 recv 682922, miss 3
8.8.8.7 <-> 9.9.9.7: send 682925 recv 682922, miss 3
8.8.8.8 <-> 9.9.9.8: send 682925 recv 682922, miss 3
8.8.8.9 <-> 9.9.9.9: send 682925 recv 682922, miss 3
8.8.8.10 <-> 9.9.9.10: send 682925 recv 682922, miss 3
Total(showing) tasks: 10
Total(showing) Sent:6829250 Recv:6829229, Miss:21
Gloabl Sent:6829250 Recv:6829229, Miss:21
```

## clearping

This command clears ping stats

## shell

This command drops into system shell

## pool

This command prints internal address pool usage (gateway only)

## stop

This command stops all tunnels gracefully (e.g. sending IKE delete msg to peer)