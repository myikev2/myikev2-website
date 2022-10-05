---
title: "Quick start"
linkTitle: "Quick Start"
weight: 10
description: >
---


## Host Compute Resource Considerations For Scale Tests
Scale test means running large number of tunnels concurrently, it has higher compute resource requirement:

* CPU: IKEv2/IPsec is CPU intensive, multi-core is required; server class CPU like Intel Xeon is recommended
* Memory: this depends on number of SAs and SA's lifetime; as reference, 100k tunnel with single CHILD_SA each tunnel, requires 48G memory as client role, 20G memory as gateway role;
* Storage: running MyIKEv2 typically doesn't require significant storage; however if this could change if you have large number of tunnels AND having detail logging
* Network: make sure there is enough network I/O bandwidth between MyIKEv2 and peer for IKEv2 and data traffic; specially in case running MyIKEv2 in a VM, high performance I/O option like SR-IOV or PCI-Passthrough to NIC is recommended.

## Installation
MyIKEv2 provides a single executable binary for Linux:

1. download from [https://www.myikev2.net](https://www.myikev2.net)
1. ```gzip -d myikev2.gz```
1. ```chmod +x myikev2```
1. move it to directory of your choice;
1. run it as root


**MyIKEv2 requires root privilege to run.**

## Linux OS Setup

* MyIKEv2 requires iproute2 to run

* for scale test, you need to change following linux kernel settings:

    * increase number of open file: ```ulimit -n <xxx>```; xxx must be bigger than number of tunnels
    * increase UDP buffer size:
    ```
    sysctl -w net.core.rmem_max=26214400
    sysctl -w net.core.rmem_default=26214400
    sysctl -w net.core.wmem_max=26214400
    sysctl -w net.core.wmem_default=26214400
    ```
    * increase NIC TX/RX queue size:
    ```
    ethtool -G <interface-name> tx <max-value> 
    ethtool -G <interface-name> rx <max-value>

    note: <max-value> could be obtained via command `ethtool -g <inteface-name>`
    ```
    * use biggest MTU available for network link between MyIKEv2 and peer (unless you want to test fragmentation)

* since v1.2, MyIKEv2 require libpcap to run; if libpcap is already installed, but you still get error msg like " error while loading shared libraries: libpcap.so.x.y: cannot open shared object file: No such file or directory", then just create a symbol link "libpcap.so.x.y" to installed libpcap.so file

* since v1.3, eap-snoop requires [eapol_test](https://w1.fi/wpa_supplicant/) executive 


## Test Setup
For each test case:

 1. create a MyIKEv2 [setup file](../setupfile/), which contains the all configurations needed to run MyIKEv2; ```myikev2 default -f <setupfilename>``` generate a setup file with default value, which could be used as starting point;
 1. use command ```myikev2 exec -f <setup_file_name>``` to run the test; for details of CLI commands, refer to [CLI Usage](../cliusage)
    * adding `-i` parameter like ```myikev2 exec -i -f <setup_file_name>``` will launch the [interactive shell](../shell/), where MyIKEv2 shell command could used to monitor the running test. 

Note: before running IKEv2 and create IPsec tunnel, if ```-flush=false``` is not specified, MyIKEv2 does following to facilitate test based on the settings in setup file:

* flush and add following address on specified interface ```bindifname```:
  * specified tunnel address (based on ```startclntaddr``` , ```numberoftunnels```, ```mobike``` and ```mobikeaddrpertunnel```) 
  * ```bindifaddr```
* flush address on interface lo, then add `127.0.0.1/32` and `::1/128` 
* flush the ip xfrm state/policy
* flush route table 330
* create a static route in main route table if "staticroute" is specified in the setup file

note: MyIKEv2 will also add following two ip xfrm polices before executing a test setup:

* ```ip xfrm policy update proto udp sport 500 dport 500 dir out priority 1```
* ```ip xfrm policy update proto udp sport 500 dport 500 dir in priority 1```

but if other tasks needed for the tests, user could write a shell script to do any setup/clean task.

### Create Setup File

* setup is a text file in YAML format
* read [setup file](../setupfile/) for documentation of all settings in setup file
* setting with default value could be omitted in setup file

### Example Setup File

see [Example Setup](../examplesetup/)


## Multiple Test Instance 

Beside the running single instance as described above, MyIKEv2 also supports test setup uses multiple instances, running on a single or multiple servers, see [controller doc](../DaemonAndController/) for details