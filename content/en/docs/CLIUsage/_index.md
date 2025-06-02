---
title: "CLI Usage"
linkTitle: "CLI Usage"
weight: 40
description: >
  MyIKEv2 CLI Usage
---


MyIKEv2 is command line based IKEv2/IPsec testing tool, it has following CLI commands:

```
MyIKEv2, an IKEv2/IPsec testing tool; Ver 5.0
https://www.myikev2.net
=======================
  = cli: connect to a myikev2 instance's interactive CLI
    --svr:
        default:127.0.0.1:12330
  = compact: compact a setup file, remove options with default value
    -c, --check: check setup file
        default:false
    -i, --input: input setup file
    -o, --output: output file
        default:myikev2_compact.setup
  = completion: Generate the autocompletion script for the specified shell
    = bash: Generate the autocompletion script for bash
      --no-descriptions: disable completion descriptions
        default:false
    = fish: Generate the autocompletion script for fish
      --no-descriptions: disable completion descriptions
        default:false
    = zsh: Generate the autocompletion script for zsh
      --no-descriptions: disable completion descriptions
        default:false
  = control: myikev2 controller
    = cli: enter myikev2 controller interactive CLI
      -c, --configfile: config file for the controller
        default:/etc/myikev2_controller.conf
      --lf: log file path
        default:myikev2_controller.log
      -o, --override: override existing test instance
        default:true
    = example: create an example recipe file
  = createpki: creates x.509v3 CA/EE certficates/keys
    --cadir: CA certs folder
    --cakeytype: the CA key type/curve and key length, like rsa:2048, ecdsa:p-384 or ed25519
        default:rsa:4096
    -l, --calvl: length of CA chain
        default:1
    --caprefix: prefix to CA cert's subject's CN
    -c, --count: number of EE cert/key pairs
        default:1
    --eedir: EE certs folder
    --eekeytype: the EE key type/curve and key length, like rsa:2048, ecdsa:p-384 or ed25519
        default:rsa:2048
    --eeprefix: prefix to EE cert's subject's CN, template when contains '&d' and '&d' will be replaced with index
  = daemon: start myikev2 test daemon
    --lf: log file path
        default:myikev2_daemon.log
    --listen: listening address:port
        default:0.0.0.0:12240
    --loglvl: logging level
        default:2
    -p, --profiling: enable profiling
        default:false
  = default: export default setup or freeradius user file
    = freeradius:
      -c, --exportcount: the number of credentials
        default:10
      -f, --exportfile: export file name
        default:users
      --temp: freeradius user config template
        default:bob&d    Cleartext-Password := "bob"
    = setup:
      -f, --exportfile: export file name
        default:default_myikev2.setup
  = echosvr: start UDP echo server
    --count: number of listening address
        default:1
    --port: listening port
        default:9922
    --startip: starting listening addr
    --step: step
        default:1
  = exec: execute a myikev2 test setup file
    --crlf: crash log file path
    -p, --enableprofile: enable profiling, dev use only
        default:false
    --flush: flush interface and xfrm states at the begining
        default:true
    -i, --invokeshell: enable interactive CLI
        default:false
    --lf: log file path
        default:myikev2.log
    -l, --licfile: license file path
        default:/etc/myikev2.lic
    -f, --setupfile: test setup file name
    -j, --usejsonresult: result formated as json
        default:false
    --verify: verify setup file only without executing
        default:false
  = help: Help about any command
  = summaryhelp: help in summary
    -h, --help: help for summaryhelp
        default:false


```

## myikev2 exec 

`myikev2 exec` execute the test [setup file](../setupfile/) specified by `-f <setup_file_path>`; test setup file is a single YAML file that defines the test, see [setup file](../setupfile/) for details.

`-l <license_file_path>` specifies an alternative location for [license file](../licensefile/) than default location (/etc/myikev2.lic); without a valid license file, MyIKEv2 will run in trial mode;

`--lf <log_file_path>` specifies log file path; default is `myikev2.log` at current directory.

With `-i` , an [interactive shell](../interactivecli/) will be opened after the setup file is loaded, which allows user to monitor running test;

With `-j`, the tunnel creation result will be printed in JSON format

With `--flush false`, MyIKEv2 will not flush the binding-interface and XFRM states/policy

`-p` is used for collecting MyIKEv2's running information, used for troubleshoot MyIKEv2 itself

`--crlf` specifies crash log file

### Example 

* `myikev2 exec -f testcase1.setup` : run test as defined in file `testcase1.setup`
* `myikev2 exec -f testcase1.setup -i --lf /var/log/testcase1.log` : run test as defined in file `testcase1.setup`, open the interactive shell, and log to `/var/log/testcase1.log`


## myikev2 createpki

`myikev2 createpki` creates PKI keys/certificates in batch, which could be used for IKEv2 authentication testing;

Basic usage is `myikev2 createpki -c <number_of key/cert> --cadir <CA_certs_dirname> --eedir <EE_certs_dirname>`, which will create following files:

* under cadir:
    * rootca.cert: Root CA certificate
    * rootca.key: Root CA's key
    * sub-lv-1_myikev2.cert: a sub-CA's certificate, signed by Root CA
    * sub-lv-1_myikev2.key: sub-CA's key

* under eedir:
    * `ee-<x>_myikev2.cert`: A End Entitiy certificate, signed by sub-lv-1_myikev2,  x is from 0 to number speicifed by `-c`
    * `ee-<x>_myikev2.key`: the key of corresponding certificate 

* note: the type of above cert/key are by default RSA

`--keytype {rsa:<keylen>|ecdsa:<curve>|ed25519}` is used to specify the type of key

`-l <number>` is used to specify the number of sub-CA in the CA chain, by default is 1, could be 0;

### customize EE certificate subject CommonName
Generated EE certificate subject's CN could be customized via `--eeprefix <str>`:
- if `<str>` contains `&d`, then it is used as template for CN, and `&d` will be replaced by the index, for example: with `--eeprefix client-&d`, the `ee-19_myikev2.cert`'s subject CN would be `client-19`
- if `<str>` doesn't contain `&d`, then it is used as prefix, followed by `ee-<x>_myikev2`, for example: with `--eeprefix client-`, the `ee-19_myikev2.cert`'s subject CN would be `client-ee-19_myikev2`

## myikev2 default

`myikev2 default setup -f <filename>` export a default setup file to `<filename>`, which could be served as starting point of a new setup file.

`myikev2 default freeradius -f <filename> -c <count> --temp <template_string>` export a freeradius user config file with number of `<count>` entries, by using a template string; the `&d` in template string will be replaced by an increasing number start from zero; for example `myikev2 default freeradius -f users -c 3 --temp  "bob&d    Cleartext-Password := \"bob\""` will export a `users` file with following content:

```
bob0    Cleartext-Password := "bob"
bob1    Cleartext-Password := "bob"
bob2    Cleartext-Password := "bob"
```

## myikev2 daemon

run MyIKEv2 as daemon, which could be controlled by a controller, see [controller doc](../Controller/) for details

## myikev2 control cli

Enter MyIKEv2 controller interactive CLI, see [controller doc](../Controller/) for details

## myikev2 control example

creates example controller configuration and recipe files, see [controller doc](../Controller/) for details

## myikev2 echosvr

run MyIKEv2 echo server, see [ping doc](../Ping/) for details

## Shell Completion

myikev2 supports auto completion for following shells:

- bash
- fish
- zsh 

the shell completion script could be generated by `myikev2 completion <shell>`; use `myikev2 completion <shell> -h` to see instructions