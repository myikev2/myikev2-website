---
title: "EAP Authentication"
linkTitle: "EAP Authentication"
weight: 90
description: >
  MyIKEv2 EAP authentication implementations.
--- 

MyIKEv2 doesn't support any EAP method directly, however it support IKEv2 EAP authentication via one of following methods:

* Client role
  * eap-file
  * eap-snoop
* Gateway role
  * EAP authentication via RADIUS server

## Client Role

### eap-file
eap-file works like following:

* User need to obtain a pcap file that contains packets of a successful RADIUS EAP authentication; 
* MyIKEv2 will function as both IKEv2 EAP client and a RADIUS server as following
```
myikev2(As IKEv2 EAP client/peer) --- DUT (as EAP Authenticator ) --- myikev2(as RADIUS EAP server)
```
so this means DUT must enable EAP RADIUS feature as defined in RFC3579.


* MyIKEv2 will extract the EAP-Message from each RADIUS packet in pcap file, and pass it through DUT via standard IKEv2 EAP authentication procedure.
  * the EAP-Message in request message in pcap file is used by MyIKEv2 as IKEv2 EAP payload to DUT
  * the response message in pcap file is used by MyIKEv2 radius server to respond to DUT


### Setup eap-file
1. Create or obtain a pcap file contains one successful radius authentication session for the EAP method you need to test. one way to create a such pcap file is to use [eapol_test](https://w1.fi/wpa_supplicant/) and [freeradius](http://freeradius.org/) (freeradius source contains eapol_test).

1. Configure following options in "ikeconf" section of MyIKEv2 setup file:

* authownmethod: set this to "eap" or "eap-only"
* eapimplementaion: set this to "eap-file"
* authpeermethod: the method used to authentication peer
* eapfile: the path to the pcap file
* eapradiusss: radius shared secret of eapfile; DUT also needs to use this as radius share secret
* eapradiussvr: the listening address of radius server; DUT need to be configured to use this as radius server
* eapradiusid: the radius attribute type in message sent by DUT to radius server that myikev2 radius server uses to **uniquely** identify a radius auth session in access-request; 
  * it has to be unique across session, could be e.g, "44" (acct-session-id) or "31"(calling-station-id); 
  * this is NOT the attribute in RADIUS request message of the pcap file, this is from DUT.
* myid: it might be necessary to set this to EAP identity

#### Notes

* for RADIUS **request** messages in the pcap file, only EAP-Message attribute is used, other are ignored.
* All tunnels define in a setupfile uses same EAP messages from pcap, so they all derive same MSK
* In case of a MyIKEv2 client with eap-file config inter-op with a MyIKEv2 gateway, make sure the gateway uses same eapradiusid as the client
* In case of scale test, the `setupinterval` can't be too small, as a rule of thumb should be >=100ms


### eap-snoop
eap-snoop works like following:

* MyIKEv2 runs `eapol_test` as EAP supplicant
* MyIKEv2 snoop/intercept the EAP message between `eapol_test` and IKEv2 peer
* The actual EAP authentication is between `eapol_test` and IKEv2 peer

#### setup eap-snoop
Configure following options in "ikeconf" section of MyIKEv2 setup file:

* authownmethod: set this to "eap"
* eapimplementaion: set this to "eap-snoop"
* authpeermethod: the method used to authentication peer
* eapoltestpath: the path point to binary of eapol_test
* eapoltestconf: the config for eapol_test; note this could be a multi-line string in YAML format; following is an example for EAP-TLS:

```
  eapoltestconf: |
    network={
        key_mgmt=WPA-EAP
        eap=TLS
        identity="ee-0_myikev2"
        ca_cert="/root/certs/rsa/rootca.cert"
        client_cert="/root/certs/rsa/ee-0_myikev2.cert"
        private_key="/root/certs/rsa/ee-0_myikev2.key"
        private_key_passwd="whatever"
    }
```
* myid: it is might be necessary to set this to EAP identity

#### Unique EAP Credential
"&d" in eapoltestconf will be replaced by tunnel index during runtime; for example "user-&d" will become "user-0" for first tunnel, "user-1" for 2nd tunnel..so on.

This could be used for having unique EAP credentials for each tunnel. 

As a related feature, ```myikev2 default -t fr -frtemp <user_config_template_string> -c <count>``` command could be used to generate a simple freeradius users config file, which could be used for gateway side; 

also setting myid to "eapol-test-conf" will change IDi to the identity value in eapoltestconf.


#### Notes

* leap is not supported with eap-snoop

#### Example setup file for eap-snoop
This setup file uses EAP-TLS:

```
runningtime: 30m
abortonerr: false
numberoftunnels: 10
loglevel: 3
startclntaddr: "11.1.0.2/8"
peeraddr: "11.1.0.1"
bindifname: "enp0s10"
bindifaddr: "11.254.254.254/8"
staticroute: ""
setupinterval: 100ms
owntsaddrincrease: 0
peertsaddrincrease: 0
ikeconf:
  dhgrpid: 14
  ikeintegrityalg: sha1-96
  ikeencyptalg: aes-cbc:128
  ikeprfalg: sha1
  authpeermethod: psk
  authownmethod: eap
  eapimplementaion: eap-snoop
  eapoltestpath: /root/eapol_test
  eapoltestconf: |
    network={
        key_mgmt=WPA-EAP
        eap=TLS
        identity="ee-0_myikev2"
        ca_cert="/root/certs/rsa/rootca.cert"
        client_cert="/root/certs/rsa/ee-0_myikev2.cert"
        private_key="/root/certs/rsa/ee-0_myikev2.key"
        private_key_passwd="whatever"
    }
  eapfile: ""
  eapradiusss: ""
  eapradiussvr: ""
  eapradiusid: 31
  myid: "ee-0_myikev2"
  cadir: ""
  eedir: ""
  dshashalg: sha256
  usersapss: false
  psk: "mypsk123"
  initiatedpd: true
  forcedpd: false
  dpdinterval: 30s
  ikelifetime: 10m0s
  margintime: 60s
  installfastpath: false
  keepchildhistory: false
  keepikehistory: false
  ratunnel: true
  enablenatt: false
  nattkeepaliveinterval: 0s
  ikev2msgmaxsize: 0
  childlist:
  - integrityalg: sha1-96
    encalg: aes-cbc:128
    protocol: esp
    lifetime: 5m0s
    esn: false
    pfsenabled: true
    pfsgrpid: 14
    replaywindowsize: 256
    ownts:
    - type: v4
      protocol: 0
      startport: 0
      endport: 65535
      startaddr: 0.0.0.0
      endaddr: 255.255.255.255
    peerts:
    - type: v4
      protocol: 0
      startport: 0
      endport: 65535
      startaddr: 0.0.0.0
      endaddr: 255.255.255.255

```

#### Build eapol_test

1. download wpa_supplicant source from https://w1.fi/releases/wpa_supplicant-2.7.tar.gz
1. tar xvf wpa_supplicant-2.7.tar.gz
1. cd wpa_supplicant-2.7/wpa_supplicant
1. wget https://raw.githubusercontent.com/FreeRADIUS/freeradius-server/master/scripts/travis/eapol_test/config_linux -O .config
1. make eapol_test

## Gateway Role

As gateway, MyIKEv2 support IKEv2 EAP authentication via a RADIUS server as speicifed by RFC3579, so the actual EAP exchange is between IKEv2 peer and RADIUS server;

gateway sends EAP-Start message to radius server upon receiving first IKE_AUTH request from client.

Following options in setup file are used in this case:

* eapradiussvr: RADIUS sever address
* eapradiusss: RADIUS share secret
* eapradiusid: the radius attribute type that's used to identify the session, gateway will insert the specified radius attribute with corresponding IKE_SA's own SPI as value in access-request
* eapsendstart: if true, the gateway sends EAP-Start to radius server at the beginning of EAP exchanges; otherwise, sends EAP-ID/Response with User-Name to radius server first 

Other eap options in setup file are ignored; 


## EAP-Only
MyIKEv2 support EAP-ONLY (RFC5998), which is enabled by setting:

  * client role: ```authpeermethod: eap-only```
  * gateway role: ```authownmethod: eap-only```

with EAP-Only, if peer also supports it and choose to use it, then msg-4 (IKE_AUTH response) will not contain AUTH payload; but if peer still choose to include AUTH payload in msg-4, then MyIKEv2 will verify it and fail tunnel setup if verification failed;