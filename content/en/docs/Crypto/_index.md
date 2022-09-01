---
title: "Crypto Algorithms"
linkTitle: "Crypto Algorithms"
weight: 80
description: >
  Crypto algorithms supported by MyIKEv2
---

This file list all supported MyIKEv2 crypto and its value in the test setup file

## IKE SA Encryption
* ```3des```
* ```aes-cbc:<keylen>```  AES-CBC; when configure this in setup file, a key length suffix is also needed; e.g. aes-cbc:128 means AES-CBC with 128bit key length
* ```aes-gcm-16:<keylen>``` : AES-GCM with 16 bytes authentication tag
* ```aes-gcm-12:<keylen>``` : AES-GCM with 12 bytes authentication tag
* ```chacha20-poly1305```

## IKE SA Integrity
* ```md5-96``` : MD5
* ```sha1-96``` : SHA1
* ```sha256``` : SHA256
* ```sha384``` : SHA384
* ```sha512``` : SHA512

## IKE SA PRF
* ```md5``` : MD5
* ```sha1``` : SHA1
* ```sha256``` : SHA256
* ```sha384``` : SHA384
* ```sha512``` : SHA512

## ESP/Fastpath Encryption
* ```null```
* ```3des```
* ```aes-cbc:<keylen>```  AES-CBC; when configure this in setup file, a key length suffix is also needed; e.g. aes-cbc:128 means AES-CBC with 128bit key length
* ```aes-gcm-16:<keylen>``` : AES-GCM with 16 bytes authentication tag
* ```aes-gcm-12:<keylen>``` : AES-GCM with 12 bytes authentication tag
* ```chacha20-poly1305```

## Diffie-Hellman Group
1/2/5/14/15/16/17/18/19/20/21/31

## PKI Key Type
* RSA
    * with ```myikev2 createpki``` generated key:
        * CA: 4096bit
        * EE: 2048bit
    * Other source:
        * any
* ECDSA with following curves:
    * with ```myikev2 createpki``` generated key:
        * CA: P521
        * EE: P384
    * Other source:
        * P224/P256/P384/P521
* Ed25519 

