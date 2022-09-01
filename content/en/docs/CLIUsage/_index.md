---
title: "CLI Usage"
linkTitle: "CLI Usage"
weight: 40
description: >
  MyIKEv2 CLI Usage
---


MyIKEv2 is command line based IKEv2/IPsec testing tool, it has following CLI commands:

```
MyIKEv2, an IKEv2/IPsec test tool; Ver 4.0
https://www.myikev2.net
======================
flag provided but not defined: -?
exec: command to execute a setup file
  -crlf string
        specify the crash log file
  -f string
        config file name
  -flush
        flush interface and xfrm states at the begining (default true)
  -i    enable interactive CLI
  -j    result formated as json
  -l string
        license file name (default "/etc/myikev2.lic")
  -lf string
        specify the log file (default "myikev2.log")
  -p    enable profiling

cli: connect to a myikev2 interactive CLI
  -port uint
        remote api svr listening port (default 12330)
  -svr string
        remote api svr listening address (default "127.0.0.1")

createpki: command to create CA and EE cert/keys
  -c int
        number of EE cert/key pairs
  -cadir string
        directory for CA
  -cakeytype string
        the CA key type/curve and key length, like rsa:2048, ecdsa:p-384 or ed25519 (default "rsa:4096")
  -eedir string
        directory for EE cert/key
  -eekeytype string
        the EE key type/curve and key length, like rsa:2048, ecdsa:p-384 or ed25519 (default "rsa:2048")
  -l int
        the length of CA chain (default 1)

default: export default setup or fr users file
  -c uint
        the number of credentials (default 10)
  -f string
        export file name
  -frtemp string
        fr user config template (default "bob&d    Cleartext-Password := \"bob\"")
  -t string
        export type, conf or fr (default "conf")


daemon: start myikev2 test daemon
  -ip string
        listening addr of daemon (default "0.0.0.0")
  -lf string
        log file (default "myikev2_daemon.log")
  -loglvl uint
        logging level (default 2)
  -p    enable profiling
  -port uint
        listening port of daemon (default 12240)

control cli: enter myikev2 controller interactive CLI
  -c string
        config file for controller (default "/etc/myikev2_controller.conf")
  -o    override existing test instance (default true)

control example: create an example recipe file
```

## myikev2 exec 

`myikev2 exec` execute the test [setup file](../setupfile/) specified by `-f <setup_file_path>`; test setup file is a single YAML file that defines the test, see [setup file](../setupfile/) for details.

`-l <license_file_path>` specifies an alternative location for [license file](../licensefile/) than default location (/etc/myikev2.lic); without a valid license file, MyIKEv2 will run in trial mode;

`-lf <log_file_path>` specifies log file path; default is `myikev2.log` at current directory.

With `-i` , an [interactive shell](../interactivecli/) will be opened after the setup file is loaded, which allows user to monitor running test;

With `-j`, the tunnel creation result will be printed in JSON format

With `-flush false`, MyIKEv2 will not flush the binding-interface and XFRM states/policy

`-p` is used for collecting MyIKEv2's running information, used for troubleshoot MyIKEv2 itself

`-crlf` specifies crash log file

### Example 

* `myikev2 exec -f testcase1.setup` : run test as defined in file `testcase1.setup`
* `myikev2 exec -f testcase1.setup -i -lf /var/log/testcase1.log` : run test as defined in file `testcase1.setup`, open the interactive shell, and log to `/var/log/testcase1.log`


## myikev2 createpki

`myikev2 createpki` creates PKI keys/certificates in batch, which could be used for IKEv2 authentication testing;

Basic usage is `myikev2 createpki -c <number_of key/cert> -cadir <CA_certs_dirname> -eedir <EE_certs_dirname>`, which will create following files:

* under cadir:
    * rootca.cert: Root CA certificate
    * rootca.key: Root CA's key
    * sub-lv-1_myikev2.cert: a sub-CA's certificate, signed by Root CA
    * sub-lv-1_myikev2.key: sub-CA's key

* under eedir:
    * `ee-<x>_myikev2.cert`: A End Entitiy certificate, signed by sub-lv-1_myikev2,  x is from 0 to number speicifed by `-c`
    * `ee-<x>_myikev2.key`: the key of corresponding certificate 

* note: the type of above cert/key are by default RSA

`-keytype {rsa|ecdsa|ed25519}` is used to specify the type of key

`-l <number>` is used to specify the number of sub-CA in the CA chain, by default is 1, could be 0;

## myikev2 default

`myikev2 default -t conf -f <filename>` export a default setup file to `<filename>`, which could be served as starting point of a new setup file.

`myikev2 default -t fr -f <filename> -c <count> -frtemp <template_string>` export a freeradius user config file with number of `<count>` entries, by using a template string; the `&d` in template string will be replaced by an increasing number start from zero; for example `myikev2 default -t fr -f users -c 3 -frtemp  "bob&d    Cleartext-Password := \"bob\""` will export a `users` file with following content:

```
bob0    Cleartext-Password := "bob"
bob1    Cleartext-Password := "bob"
bob2    Cleartext-Password := "bob"
```

## myikev2 daemon

run MyIKEv2 as daemon, which could be controlled by a controller, see [controller doc](../DaemonAndController/) for details

## myikev2 control cli

Enter MyIKEv2 controller interactive CLI, see [controller doc](../DaemonAndController/) for details

## myikev2 control example

creates example controller configuration and recipe files, see [controller doc](../DaemonAndController/) for details