---
title: "MyIKEv2 Daemon & Controller"
linkTitle: "MyIKEv2 Daemon & Controller"
weight: 130
description: >
  scale out MyIKEv2 to multiple instance, orchestrated by a controller.
---

Some test cases could require running multiple MyIKEv2 instances on one or multiple servers; one example is large scale test, where multiple MyIKEv2 test instances on multiple server are needed;  MyIKEv2 supports such test case in a simple and orchestrated way by using MyIKEv2 daemon and controller; 
```   
                 --- SVR1[daemon --> MyIKEv2_Instance_1,MyIKEv2_Instance_2 ...]
controller ---- |--- SVR2[daemon --> MyIKEv2_Instance_3,MyIKEv2_Instance_4 ...]
                 --- SVR3[daemon --> MyIKEv2_Instance_5,MyIKEv2_Instance_6 ...] 
```
- MyIKEv2 Daemon: a daemon process running on a given server, it manages (like creating/stoping ..etc) MyIKEv2 instances by accepting API calls from the MyIKEv2 controller; there is only one daemon process per server;
- MyIKEv2 Controller: the controller to control all daemons, it manages the test cases by by controlling the daemon processes on each server; only one controller globally is required; 
- Recipe: a YAML file specified by user, which defines the test case that could be launched by using the interactive CLI of controller; each recipe defines one or multiple MyIKEv2 instances;
- MyIKEv2 Instance: a instance is a "myikev2 exec" instance running in its own Linux network namespace, which means each instance has its own interfaces, route table and xfrm policy/states, the instance and its namespace is created by the daemon 

## Network Provisioning  

Before using this feature, there are some network plumbing work to do, which include provision of:
- Management Networking: networking for controller <-> daemon message, and daemon <-> instances message
- Data Networking: networking for actual IPsec/IKE packets of MyIKEv2 instances 

### Management Networking

* For each server, user needs to specify a IPv6 prefix in control's config file, and each myikev2 instance will get a management IP address within that prefix. 
    * user need to provision routing between controller and each server so that these management prefix are reachable from controller
    * auto assigning could be disable by set `autoassignmgmtip` to false in controller's config file; if disabled, then each instance's managment is specified by `apilistenaddr` in instance's setup file. 
* On each server, daemon will create a bridge: myikev2mgmtbrg
* For each myikev2 instance, a pair of veth interfaces are created by daemon, one end is attached to myikev2mgmtbrg, the other end is assigned to the instance's namespace; daemon also attaches the instance's management address to the veth interface in the namespace ;
* daemon creates a default route in the namespace with nexthop as the veth if
* daemon creates a host route for each instance in base namespace with nexthop as the myikev2mgmtbrg
* there is no address needed on bridge if and veth if in the base

### Data Networking

For actual IKEv2/IPsec traffic, user need to have an interface for each namespace that provides connectivity needed for the test case; the interface is specified by `bindifname` in the MyIKEv2 setup file; daemon will move the interface into the instance's namespace when creating the MyIKEv2 instance.

## Configuration 

There are following type of configuration files are needed:
- Controller's config file
- Recipe: one per test case
- Setup: one per MyIKEv2 instance in the recipe

### Controller Configuration
Controller configuration file is a YAML file contains following parts:
- `daemonlist`: a dictionary, key is the svr/daemon name, value is a struct:
    - `daemonaddr`: daemon's listening address
	- `daemonport`: daemon's listening port
	- `sshaddr`: server's SSH address
	- `sshport`: server's SSH port     
	- `sshuser`: server's SSH username 
	- `sshpass`: server's SSH password
	- `sshkeypath`: user ssh private key path
	- `mgmtaddrprefix`: the IPv6 prefix for auto assigning instance's management IP
- `varlist`: a dictionary, key is the variable name, value is the variable value; the variable defined here could be used in command strings (like `setupcmds` and `cleancmds`) in recipe; 
- `autoassignmgmtip`: boolean, true or false; 

Following is an example:
```
daemonlist:
  svr1:
    daemonaddr: 1.1.1.1
    daemonport: 12240
    sshaddr: 1.1.1.1
    sshport: 22
    sshuser: root
    sshpass: passwd123
    sshkeypath: ""
    mgmtaddrprefix: 2001:dead:1::/64
  svr2:
    daemonaddr: 2.2.2.2
    daemonport: 12240
    sshaddr: 2.2.2.2
    sshport: 22
    sshuser: root
    sshpass: passwd123
    sshkeypath: ""
    mgmtaddrprefix: 2001:dead:2::/64
varlist:
  '%DIRPREFIX%': /root/testcases
autoassignmgmtip: false
```
### Recipe 

Recipe is a YAML file defines a test case, contains following:
- `name`: the name of the test case
- `setupcmds`: a list of shell commands run before running the test case, each is a struct contains following:
   -  `daemonname`: name of daemon that command will be running on
   - `cmds`: a string contains one or multiple shell commands, separated by `;`
- `cleancmds`: a list of shell command run after test case ended, same structure as `setupcmds`
- `instancelist`: a dictionary defines all test instances, key is the instance ID, an integer; value a struct contains:
    - `name`: name of instance 
    - `daemonname`: name of daemon that the test instance will be running on
    - `waitinterval`: wait amount of time before starting the the instance
    - `myikev2`: a struct specifies the instance is a MyIKEv2 instance, contains:
        - `setupath`: the path to the MyIKEv2 setup file
        - `logdir`: the path to save the MyIKEv2 log
    - `other`: a struct specifies a non-MyIKEv2 instance, could be used to start a 3rd party application, like strongswan
        - `setupcmds`: setup commands, a string contains one or multiple shell commands, separated by `;`
        - `upcmds`:   startup commands, a string contains one or multiple shell commands, separated by `;`   
        - `destroycmds`: stop commands, a string contains one or multiple shell commands, separated by `;`   
        - `dataif`: the interface name of data traffic     
        - `dataifaddr`: the IP address to attached to the `dataif`
    - note: for a given instance, either `myikev2` or `other` needs to specified, but not both

following is an example:
```
name: Example MyIKEv2 Recipe
setupcmds:
- daemonname: svr1
  cmds: cmd-1;cmd-2
- daemonname: svr2
  cmds: cmd-3;cmd-4
cleancmds: []
instancelist:
  1:
    name: test-1_client
    daemonname: svr1
    waitinterval: 3s
    myikev2:
      setuppath: testc.setup
      logdir: ""
    other: null
  2:
    name: test-1_gateway
    daemonname: svr2
    waitinterval: 3s
    myikev2: null
    other:
      setupcmds: ""
      upcmds: ipsec up
      destroycmds: ""
      dataif: eth1
      dataifaddr: 172.16.100.1/24
```

## Usage
1. create all the configuration files on controller server
1. provision networking as described above
1. run `myikev2 daemon --listen <addr:port>` on each daemon server
1. run `myikev2 control cli -c <config_file>` on controller server to start the interactive controller CLI
1. use the controller CLI to manage the test case

## Controller CLI

Controller interactive CLI provides following commands:
- `def -f <recipe_file_name>`: start a test case specified by using a recipe file 
- `list [-d <daemon_name>]`: list launched test instance 
- `cli -t <instance_name>`: connect to a MyIKEv2 instance's CLI 
- `shell`: drop into a system shell
- `clearping [-t <instance_name>]`: reset specified MyIKEv2 instance's ping stats; if instance_name is not specified, then clear all MyIKEv2 instance's ping stats
= `stop [-g <true|false>] -t <instance_name>`: stop a specified instance; gracefully stop a MyIKEv2 instance when `-g=true` 
