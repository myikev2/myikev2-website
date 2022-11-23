---
title: "Default Setup File with Comments"
linkTitle: "Default Setup File with Comments"
weight: 30
description: >
  MyIKEv2 setup explained
---

MyIKEv2 uses a YAML file as the setup file for all its configurations; 
A default setup file could be generated via command "myikev2 default";

Following is the default setup file with comments describe each option:

```
# This is a test setup file with default value for MyIKEv2, it is in YAML;
# see comment on top of each option for description
# note: some parameters are only applied to a specific role, as specified in comments; 
#       without it, it means the parameter apply to both roles

# role of the MyIKEv2
# client: tunnel initiator
# gateway: tunnel responder
role: client

# The description of the test setup, displayed at the beggging of test
desc: ""

# the duration of the test will run for
# the format is like 2h (2 hours),3m (3 minutes),5s (5 seconds)
# zero means test will run forever (10 years)
runningtime: 0s

# client role: the number of tunnels MyIKEv2 will try to create 
# gateway role: the number of tunnel expect peer to create
numberoftunnels: 10

# log level: file log level; 
# note: this is separate from STDOUT log with interactive CLI 
# 1 means only critical msg, 
# 2 means error plus previous level, 
# 3 means warning plus previous level, 
# 4 means information msg plus previous level, 
# 5 means debug (include decoded pkt output) plus previous level
loglevel: 2

# the keyword used to filter out log message
# only messages contains logkeyword will be logged in the log file
# empty means no filter
logkeyword: ""

# the cap size of the log file, in MB, once size exceeds the cap
# current log file will be renamed with a suffix ".prev"
# so it means myikev2 will have maximum two log files, 
# each size at logfilesizecap
# this also applies if loginmem is true
# 0 means no cap
logfilesizecap: 1000

# if true, keep log message in memory,
# only write into file upon finish
# set this true while doing scale test could save some resource on logging
loginmem: false

# if ture, myikev2 will abort at 1st error;
abortonerr: false

# client role only
# starting tunnel address used by MyIKEv2
# each following tunnel will be assigned with +1 of previous tunnel address
# this could be either IPv4 or IPv6 address with prefix length. e.g. x.x.x.x/y
# for example if numberoftunnels is 3 and startclntaddr is 1.1.1.1/24,
# then 1st tunnel gets 1.1.1.1/24, 2nd gets 1.1.1.2/24, 3rd get 1.1.1.3/24
# madantory, can't be empty
startclntaddr: ""

# Enable MOBIKE(RFC4555)
mobike: false


# client role only
# the number of own tunnel address per tunnel
mobikeaddrpertunnel: 2

# gateway role only
# the number of own tunnel address for the gateway
mobikeaddrpergw: 2

# client role only
# the amount of time MyIKEv2 wait before change to next address
mobikeiplifetime: 5m0s

# client role only
# how MyIKEv2 change the address using MOBIKE
# own-only: only change own tunnel address
# peer-only: only change peer's tunnel address
# both: change both own and peer's tunnel address
mobikechangeaddrtype: own-only

# client role only
# gateway will accept incoming requests regardless its address
# IKEv2 peer's tunnel address, all tunnels created will use this peer address
# madantory, can't be empty
peeraddr: ""

# the name of Linux interface to which MyIKEv2's own tunnel addresses bind
# note: you should use a dedicate interface,
# since this interface could get flushed during initiatlization
# madantory, can't be empty
bindifname: ""

# for gateway role: this is the local tunnel address
# for client role:
# an addtional address to add on the binding interface,
# which is beside MyIKEv2's own tunnel addresses;
# this could be used to faciliate certain routing setup
# this could be either IPv4 or IPv6 address with prefix length.
# e.g. x.x.x.x/y
bindifaddr: ""

# addtioanl static route to be created to faciliate certain routing requirements;
# it should be a "ip route replace ..." command
# use "ip route replace" instead of "ip route add" to avoid failure of repeated running same command
staticroute: ""

# client role only
# the time MyIKEv2 wait before creating next tunnel
setupinterval: 100ms

# client role only
# the interval between tunnel creation retry
tunnelretryinterval: 10s

# client role only
# the max tries for a specific tunnel creation
tunnelmaxretry: 3

# client role only
# for each tunnel,
# the step MyIKEv2 will increate for address in ownts of childlist section 
# for example, with owntsaddrincrease:3,
# if startaddr and endaddr in ownts config are both 192.168.1.1, 
# then 1st tunnel will use 192.168.1.1-192.168.1.1 as its own TS address range, 
# 2nd tunnel will use 192.168.1.4-192.168.1.4 as its own TS address range ...
owntsaddrincrease: 0

# client role only
# for each tunnel,
# the step MyIKEv2 will increate for address in peerts of childlist section
# see comments of owntsaddrincrease for details
peertsaddrincrease: 1

# listening address for API server
apilistenaddr: 0.0.0.0

# listening port for API server
apilistenport: 12330


# flapconf are the config for tunnel flapping, client only
flapconf:
  # enable/disable tunnel flapping
  flapping: false
  # number of tunnel flapping, must <= numberoftunnels
  # -1 means same as numberoftunnels
  numoftunnel: -1
  # the interval between two dials is a random number
  # between minflapinterval and maxflapinterval
  # minflapinterval must >= 10s
  minflapinterval: 30s
  maxflapinterval: 1m0s

# poolconf is for gateway role only
# this is the address pool from which gateway assign address to peer via IKEv2 config payload
# leave it empty means no address allocation
poolconf:
  # starting prefix of IPv4 address pool, in format of a.b.c.d/prefix_len
  v4startaddr: ""
  # a list of IPv4 DNS sever address
  v4dnslist: []
  # starting prefix of IPv6 address pool, in format Iof v6_addr/prefix_len
  v6startaddr: ""
  # a list of IPv6 DNS server address
  v6dnslist: []

ikeconf:

# gateway role only
# the number of half open IKE_SA that trigger IKEv2 cookie exchange
# a negative value disable the cookie exchange
  numoftunneltoenablecookie: -1

# Enable repeated auth (RFC4478)
  reauth: false

# gateway role only
# the amount of time peer need to do reauth
  reauthlifetime: 10s

# IKE request retry interval
  retraninterval: 10s

# the max tranmission of a IKE request
  maxretran: 4

# a list of DH groups during IKE_SA_INIT exchange
# see crypto.md for all supported crypto algorithms
  dhgrpid: 
  - 14 

# enable IKEv2 fragmentation 
  enablefragment: false

# MTU for IKEv2 fragmentation
# note: the MTU here is max size of clear IKEv2 payloads, 
# so the result IKEv2 packet after fragmentation will be bigger than this value with IKEv2 encap/encryption/IP encap overheads
  mtu: 1100

# IKEv2 message reassembly timeout 
  reassembletimeout: 30s  

# client role only
# include INITIAL_CONTACT in first IKE_AUTH request
  initialcontact: true

# a list of IKE integrity Algs
# see crypto.md for all supported crypto algorithms
  ikeintegrityalg: 
  - sha256

# a list of IKE encryption Alg
# see crypto.md for all supported crypto algorithms
  ikeencyptalg:
  - aes-cbc:128

# a list of IKE PRF Alg
# see crypto.md for all supported crypto algorithms
  ikeprfalg: 
  - sha256

# IKEv2 authentication method to autenticate peer
# support following methods:
# psk: Shared Key Message Integrity Code
# rsa: RSA Digital Signature
# ecdsa-sha256-p256, ecdsa-sha384-p384, ecdsa-sha512-p521: ECDSA auth as defined in RFC4754
# digital-signature: digital signature as defined in RFC7427
# "eap" or "eap-only": EAP authentication
  authpeermethod: psk

# IKEv2 authentication method to autenticate self 
# (a.k.a generating own AUTH payload)
# options are same as authpeermethod, 

  authownmethod: psk

# client role only
# EAP authentication implementation
# eap-snoop or eap-file
  eapimplementaion: eap-snoop

# client role only
# the config for eapol_test, used by eap-snoop
# could be multi-line YAML string
# "&d" in eapoltestconf will be replaced by tunnel index 
  eapoltestconf: ""

# client role only
# path to eapol_test binary, used by eap-snoop
  eapoltestpath: /root/gowork/src/myikev2/eapol_test

# client role only
# pcap file used for EAP RADIUS Authentication
# used by eap-file
  eapfile: ""

# gateway role:
# this is the shared secret for radius server 
# client role:
# radius share secret for the pcap file
# used by eap-file  
  eapradiusss: ""

# gateway role:
# this is the address for radius server 
# client role:
# listening address for local radius server
# used by eap-file
# for example, could be sth like "1.1.1.1:1812"
  eapradiussvr: ""

# in case of client role:
# the radius attr type DUT uses to identify a EAP RADIUS auth session
# used by eap-file
# for example, could be 31 (which is Calling-Station-Id)
# in case of gateway role, gateway will insert the specified attribute with value as corresponding's IKESA's own SPI, into access-request
  eapradiusid: 31

# gateway role only
# if true, the gateway sends EAP-Start to radius server at the beginning of EAP exchanges,
# otherwise, sends EAP-ID/Response with User-Name to radius server first
  eapsendstart: true

# local IKEv2 ID payload, supports following:
# client-src-addr: using own tunnel address, type ID_IPV4_ADDR or ID_IPV6_ADDR
# eapol-test-conf: using identity of eapol_test config
# eapusepcap: use the user-name in 1st access-request pkt in eapfile 
# cert-dn: use assigned certificate Subject, type ID_DER_ASN1_DN
# an IPv4 or IPv6 address, type ID_IPV4_ADDR or ID_IPV6_ADDR
# a RFC822 address (e.g. email addr), type ID_RFC822_ADDR
# a FQDN, type ID_FQDN 
# the default type is ID_FQDN
  myid: client-src-addr

# The path to where CA certificates are stored;
# the certificate must be in PEM format
# madantory if PKI related authentication is used
  cadir: ""

# The path to where End Entitity certificates and keys are stored;
# both certficate and key file must in clear PEM format;
# the file name of certificates/keys must be same as files that generated 
# End-Entity cert/key file name must follow following rules:
#  * cert file name must end with ".cert"
#  * key file name must end with ".key"
#  * the prefix of corresponding cert and key file name must be same, for example "ee-1_myikev2.cert" and "ee-1_myikev2.key"
# madantory if PKI related authentication is used
  eedir: ""

# Hash alg for digital signature (RFC7427) authentication
# this will be included in generated SIGNATURE_HASH_ALGORITHMS notification
  dshashalg: sha256

# enable using RSA PSS signature, default is PKCS1.5 signature
  usersapss: false

# Pre-Shared key for psk authentication
  psk: ""

# if true:
#     CERTREQ payload only include CAs whose filename's suffix contains "reqp"
#     In addtion to signing EE cert, CERT payload from CAs whose filename's suffix contains "certp"
# if false:
#     CERTREQ payload only include root CAs
#     only CERT payload from signing EE cert will be sent 
  sendingmarkedca: false

# if initiate DPD  
  initiatedpd: true

# force using DPD even peer's IKEv2 msg is received
  forcedpd: false

# DPD interval  
  dpdinterval: 30s

# IKE SA lifetime
  ikelifetime: 10m0s

# amount of time when MyIKEv2 initiate IKE_SA rekey before it expires; 
# for example if ikelifetime is 10 minutes, margintime is 2 minutes, 
# then IKE_SA rekey will be initiated at 8 minutes 
  margintime: 1m0s

#
# when set to true, MyIKEv2 will derive in-use margintime from range (0,margintime)
# 

  jitter: true

# install CHILD_SA to fastpath if true, e.g. linux kernel;
# in case IKEv2 only tests, you could choose to set this to false,
# so that consumple less resource
  installfastpath: false

# the name of interface which assigned virtual address will be attached to,
# client role only
  virtualipif: lo

# MyIKEv2 will keep rekeyed CHILD SA if true; 
# note: set this to true in scale test could consume more memory
  keepchildhistory: false

# MyIKEv2 will keep rekeyed IKE SA if true; 
# note: set this to true in scale test could consume more memory
  keepikehistory: false

# client role only
# setting true to use configuration payload to request following atttributes:
# INTERNAL_IP4_ADDRESS / INTERNAL_IP4_DNS
# INTERNAL_IP6_DNS / INTERNAL_IP6_ADDRESS
  ratunnel: false

# setting true to enable NAT-Travelsal 
  enablenatt: false

# set to true to include NO_NATS_ALLOWED notification in IKE_AUTH request and UPDATE_SA_ADDRESSES request.
disallownat: false

# the interval of sending NAT-T keepalive, setting to zero disable it 
  nattkeepaliveinterval: 0s

# if ture, enable ESP or UDP encap
  traditionalencap: true

# if true, force to use UDP encap
  tradtionalencapforceudp: false

# if ture, enable TCP encap
  tcpencap: false

# own port for TCP encap
  owntcpport: 4500

# peer port for TCP encap
  peertcpport: 4500


# a list of CHILD SA config, could be one or mulitple set of following config
  childlist:

# setting tunnelmode as false to create transport mode CHILD_SA
  - tunnelmode: true

# a list of CHILD SA integrity Algs
# see crypto.md for all supported crypto algorithms
    integrityalg: 
    - sha256

# a list of CHILD SA encryption Algs
# see crypto.md for all supported crypto algorithms
    encalg: 
    - aes-cbc:128

# CHILD SA protocol
# see crypto.md for all supported crypto algorithms
    protocol: esp

# CHILD SA lifetime
# note: CHILD_SA rekey will also use margintime as IKE SA
    lifetime: 5m0s

# whether to use ESN
    esn: false

# whether to use PFS (Perfect Forward Secrecy)
    pfsenabled: false

# a list of DH groups for PFS
    pfsgrpid: 
    - 14

# the size of anti-replay window
    replaywindowsize: 256

# if true, use configured TS for CHILD_SA rekey, otherwise uses TS of previous CHILD_SA
    useprovisionedtsforrekey: false

# Own Traffic Selector (TS) config
    ownts:

    # IP version, v4 or v6
    - type: v4

    # TS protocol, 0 means any
      protocol: 0

    # start port of the port range      
      startport: 0
      
    # end port of the port range  
      endport: 65535
    
    # start address of the address range
      startaddr: 0.0.0.0
    
    # end address of the address range
      endaddr: 255.255.255.255

# Peer Traffic Selector (TS) config   
    peerts:
    - type: v4
      protocol: 0
      startport: 0
      endport: 65535
      startaddr: 0.0.0.0
      endaddr: 255.255.255.255
# Built-in Ping test
pingconf:

# if true, the first address of negotiated TSi/TSr will be used as
# src/dst address of ICMP packet
  autoaddr: false

# type of ping, choice of icmp or udp
# in case of udp, myikev2 echosvr need to be used as the target
  ptype: icmp

# the UDP port used by UDP ping (as both src and dst)
  udpport: 9922

# if true, myikev2 will auto add ping's src address to the interface specified by virtualipif, 
# can't be enabled for client remote-access tunnel
  assignsrc: false

# ping dest of 1st tunnel; empty means disable whole ping test
  destaddr: ""

# step increase for ping dest addr of following tunnels
  deststep: 1

# ping src of 1st tunnel; empty means let OS select src 
  srcaddr: ""

# step increase for ping src addr of following tunnels
  srcstep: 1

# interval between each ping ECHO request 
  interval: 1s

# size of ping; the actual IP packet size is bigger than this
  pktlen: 64

# the percentage of max packet loss rate for a given task; 
# generate error event if rate exceeds this value
  maxlossrate: 10

# the amount of time system waits before start ping, after:
# client role: all tunnel are created
# gateway role: gateway is created 
  holdtime: 10s
  ```