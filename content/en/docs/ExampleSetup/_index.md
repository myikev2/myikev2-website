---
title: "Example Setup"
linkTitle: "Example Setup"
weight: 20
description: >
  Example setups
---


## Example setup

client (epn0s10) ------ (enp0s10) gateway

* interface: enp0s10
* Tunnel address:
    * Gateway: ```11.1.0.1/8```
    * Client: start from ```11.1.0.2/8```
* Virtual address pool on GW side (assign to client via Config Payload): 
    * prefix: ```192.168.100.1/24```
    * DNS Server address list: 8.8.8.8, 4.4.4.4
* Certificate/key directory (following locations are the directory):
    * client: 
        * CA cert: /usr/local/etc/certdb_ecdsa/client/ca
        * End-Entity cert/key: /usr/local/etc/certdb_ecdsa/client/ee
    * gateway: 
        * CA cert: /usr/local/etc/certdb_ecdsa/gw/ca
        * End-Entity cert/key: /usr/local/etc/certdb_ecdsa/gw/ee
    * All certificate and key files must be in clear PEM format
    * End-Entity cert/key file name must follow following rules:
        * cert file name must end with ".cert"
        * key file name must end with ".key"
        * the prefix of corresponding cert and key file name must be same, for example "ee-1_myikev2.cert" and "ee-1_myikev2.key"
    * MyIKEv2 could generate cert/key in batch via command "myikev2 createpki" 

## Remote-Access Tunnel with Pre-shared Key 
* Client:

```
numberoftunnels: 10
startclntaddr: "11.1.0.2/8"
peeraddr: "11.1.0.1"
bindifname: "enp0s10"
ikeconf:
  psk: "pre-shared-key"
  installfastpath: true
  ratunnel: true
  childlist:
  -
    peerts:
    -
      startaddr: 9.9.9.9
      endaddr: 9.9.9.9
```

* Gateway:

```
role: gateway
bindifname: "enp0s10"
bindifaddr: "11.1.0.1/8"
poolconf:
  v4startaddr: "192.168.100.1/24"
  v4dnslist: [8.8.8.8,4.4.4.4]
  v6startaddr: ""
  v6dnslist: []
ikeconf:
  psk: "pre-shared-key"
  ikelifetime: 20m
  installfastpath: true
  ratunnel: true
  childlist:
  -
    liftime: 10m
```

## Remote-Access Tunnel with Certificate Authentication

* Client:

```
numberoftunnels: 10
startclntaddr: "11.1.0.2/8"
peeraddr: "11.1.0.1"
bindifname: "enp0s10"
ikeconf:
  authpeermethod: digital-signature
  authownmethod: digital-signature
  cadir: "/usr/local/etc/certdb_ecdsa/client/ca"
  eedir: "/usr/local/etc/certdb_ecdsa/client/ee"
  installfastpath: true
  ratunnel: true
  childlist:
  -
    peerts:
    -
      startaddr: 9.9.9.9
      endaddr: 9.9.9.9
```

* Gateway:

```
role: gateway
bindifname: "enp0s10"
bindifaddr: "11.1.0.1/8"
poolconf:
  v4startaddr: "192.168.100.1/24"
  v4dnslist: [8.8.8.8,4.4.4.4]
  v6startaddr: ""
  v6dnslist: []
ikeconf:
  authpeermethod: digital-signature
  authownmethod: digital-signature
  cadir: "/usr/local/etc/certdb_ecdsa/gw/ca"
  eedir: "/usr/local/etc/certdb_ecdsa/gw/ee"
  installfastpath: true
  ikelifetime: 20m
  ratunnel: true
  childlist:
  -
    lifetime: 10m
```

## Remote-Access Tunnel with EAP-MD5 Authentication

* client:
    * client use eap-snoop, which uses eapol_test, see [EAP](../eap/) document for details
    * "&d" in eapoltestconf will be replace by an incrementing counter value 
    * myid is set to "eapol-test-conf", which uses the identity in eapoltestconf

```
numberoftunnels: 10
startclntaddr: "11.1.0.2/8"
peeraddr: "11.1.0.1"
bindifname: "enp0s10"
ikeconf:
  psk: "pre-shared-key"
  authpeermethod: psk
  authownmethod: eap
  eapimplementaion: eap-snoop
  eapoltestconf: |
    network={
        key_mgmt=NONE
        eap=MD5
        identity="bob&d"
        password="bob"
    }
  eapoltestpath: /root/eapol_test
  myid: "eapol-test-conf"
  installfastpath: true
  ratunnel: true
  childlist:
  -
    peerts:
    -
      startaddr: 9.9.9.9
      endaddr: 9.9.9.9
```

* gateway:

    * gateway proxy EAP exchange to external RADIUS server
    * gateway uses EAP to authentication client; while client uses psk to authenticate gateway

```
role: gateway
bindifname: "enp0s10"
bindifaddr: "11.1.0.1/8"
poolconf:
  v4startaddr: "192.168.100.1/24"
  v4dnslist: [8.8.8.8,4.4.4.4]
  v6startaddr: ""
  v6dnslist: []
ikeconf:
  psk: "pre-shared-key"
  ikelifetime: 20m
  authpeermethod: eap
  authownmethod: psk
  eapradiusss: "testing123"
  eapradiussvr: "127.0.0.1:1812"
  installfastpath: true
  ratunnel: true
  childlist:
  -
    liftime: 10m
```