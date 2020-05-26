---
layout: post
title: VMware ESXi and Vagrant
description: Network automation does not an automated network make.
summary: How Not to Automate
comments: false
---

An ESXi server can be managed via SSH, this means bypassing the graphical interface. Sometimes you may need to change configuration or require some information which the GUI doesn't provide, in which case the CLI is what you need.

Here are some ESXCLI commands:

```bash
[root@cactus-uk-far-mon:~] esxcli system version get
   Product: VMware ESXi
   Version: 6.7.0
   Build: Releasebuild-8169922
   Update: 0
   Patch: 0
```

```bash
[root@cactus-uk-far-mon:~] esxcli hardware memory get
   Physical Memory: 68363186176 Bytes
   Reliable Memory: 0 Bytes
   NUMA Node Count: 1
```   

```bash
[root@cactus-uk-far-mon:~] esxcli network nic list
Name    PCI Device    Driver  Admin Status  Link Status  Speed  Duplex  MAC Address         MTU  Description
------  ------------  ------  ------------  -----------  -----  ------  -----------------  ----  -------------------------------------------------------
vmnic0  0000:02:00.0  ntg3    Up            Up            1000  Full    20:67:7c:d9:a1:6c  1500  Broadcom Corporation NetXtreme BCM5719 Gigabit Ethernet
vmnic1  0000:02:00.1  ntg3    Up            Up            1000  Full    20:67:7c:d9:a1:6d  1500  Broadcom Corporation NetXtreme BCM5719 Gigabit Ethernet
vmnic2  0000:02:00.2  ntg3    Up            Down             0  Half    20:67:7c:d9:a1:6e  1500  Broadcom Corporation NetXtreme BCM5719 Gigabit Ethernet
vmnic3  0000:02:00.3  ntg3    Up            Down             0  Half    20:67:7c:d9:a1:6f  1500  Broadcom Corporation NetXtreme BCM5719 Gigabit Ethernet
```

```bash
[root@cactus-uk-far-mon:~] esxcli network vm list
World ID  Name         Num Ports  Networks
--------  -----------  ---------  ------------------------------
 4455488  ubuntu_gns3          1  Server Network
 4455345  gns3_vm              2  Server Network, Server Network
```

List virtual switches and port groups:

```
[root@cactus-uk-far-mon:~] esxcli network vswitch standard list
vSwitch0
   Name: vSwitch0
   Class: cswitch
   Num Ports: 4736
   Used Ports: 4
   Configured Ports: 128
   MTU: 1500
   CDP Status: listen
   Beacon Enabled: false
   Beacon Interval: 1
   Beacon Threshold: 3
   Beacon Required By: 
   Uplinks: vmnic0
   Portgroups: Management VMkernel

vSwitch1
   Name: vSwitch1
   Class: cswitch
   Num Ports: 4736
   Used Ports: 10
   Configured Ports: 1024
   MTU: 1500
   CDP Status: listen
   Beacon Enabled: false
   Beacon Interval: 1
   Beacon Threshold: 3
   Beacon Required By: 
   Uplinks: vmnic3, vmnic2, vmnic1
   Portgroups: Server Network
```   

Check the list of running VMs and display their World IDs:

```bash
[root@cactus-uk-far-mon:~] esxcli storage vmfs extent list
Volume Name  VMFS UUID                            Extent Number  Device Name                           Partition
-----------  -----------------------------------  -------------  ------------------------------------  ---------
VMs          5bc7459f-967eeb48-b3f7-20677cd9a16c              0  naa.600508b1001cf76cb54a0dec4acc6459          3
[root@cactus-uk-far-mon:~] esxcli vm process list
gns3_vm
   World ID: 4455345
   Process ID: 0
   VMX Cartel ID: 4455344
   UUID: 56 4d 12 f0 01 11 99 9e-24 51 75 dc de 5f 4b 10
   Display Name: gns3_vm
   Config File: /vmfs/volumes/5bc7459f-967eeb48-b3f7-20677cd9a16c/gns3_vm/gns3_vm.vmx

ubuntu_gns3
   World ID: 4455488
   Process ID: 0
   VMX Cartel ID: 4455487
   UUID: 56 4d 0c 5a 5c 01 56 36-6d 3b b6 0a 0a e0 46 fe
   Display Name: ubuntu_gns3
   Config File: /vmfs/volumes/5bc7459f-967eeb48-b3f7-20677cd9a16c/ubuntu_gns3/ubuntu_gns3.vmx
```

This blog has an excellent overview of esxcli commands:
https://www.nakivo.com/blog/most-useful-esxcli-esxi-shell-commands-vmware-environment/

<br/>

### Enable SSH and Console Shell

Enable SSH and the Console Shell on your ESXi server:

* ESXi-webgui/Host/Actions/Services/Enable Secure Shell (SSH)
* ESXi-webgui/Host/Actions/Services/Enable Console Shell

Now you should be able to SSH: 

```
lawrence@cacti:~$ ssh root@10.130.2.122
The time and date of this login have been sent to the system logs.

WARNING:
   All commands run on the ESXi shell are logged and may be included in
   support bundles. Do not provide passwords directly on the command line.
   Most tools can prompt for secrets or accept them from standard input.

VMware offers supported, powerful system administration tools.  Please
see www.vmware.com/go/sysadmintools for details.

The ESXi Shell can be disabled by an administrative user. See the
vSphere Security documentation for more information.
[root@cactus-uk-far-mon:~] 
```

<br/>

### Add your SSH key

According to VMWare Knowledge Base Article 1002866, the public key has to be copied to `/etc/ssh/keys-<username>/authorized_keys`:

```bash
[root@cactus-uk-far-mon:~] vi /etc/ssh/keys-root/authorized_keys
```

<br/>

### Setting static IP for the ESXi host

My ESXi server already has a static `10.130.2.122`:

```bash
[root@cactus-uk-far-mon:~] esxcli network ip interface ipv4 get
Name  IPv4 Address  IPv4 Netmask   IPv4 Broadcast  Address Type  Gateway     DHCP DNS
----  ------------  -------------  --------------  ------------  ----------  --------
vmk1  10.130.2.122  255.255.255.0  10.130.2.255    STATIC        10.130.2.1     false
```

<br/>

### Configuring a datastore

I'll be using an existing datastore which I've named 'VMs':

```bash
[root@cactus-uk-far-mon:~] esxcli storage filesystem list
Mount Point                                        Volume Name  UUID                                 Mounted  Type             Size          Free
-------------------------------------------------  -----------  -----------------------------------  -------  ------  -------------  ------------
/vmfs/volumes/5bc7459f-967eeb48-b3f7-20677cd9a16c  VMs          5bc7459f-967eeb48-b3f7-20677cd9a16c     true  VMFS-6  1192121860096  556500254720
/vmfs/volumes/ddae0775-664c51a1-87b5-6510bda38aac               ddae0775-664c51a1-87b5-6510bda38aac     true  vfat        261853184     261844992
/vmfs/volumes/5bc745a0-c86247be-18e3-20677cd9a16c               5bc745a0-c86247be-18e3-20677cd9a16c     true  vfat       4293591040    4267114496
/vmfs/volumes/5bc74599-4f2de32c-078c-20677cd9a16c               5bc74599-4f2de32c-078c-20677cd9a16c     true  vfat        299712512      80486400
/vmfs/volumes/493dce79-9720ce73-6cf3-f56441522235               493dce79-9720ce73-6cf3-f56441522235     true  vfat        261853184     113819648
```

<br/>

### Network Configuration 

I'll be using the portgroup `Server Network` on `vSwitch1` in `VLAN 0` (`vSwitch0` is the default virtual switch):

```bash
[root@cactus-uk-far-mon:~] esxcli network vswitch standard portgroup list
Name                 Virtual Switch  Active Clients  VLAN ID
-------------------  --------------  --------------  -------
Management VMkernel  vSwitch0                     1        0
Server Network       vSwitch1                     3        0
```

<br/>

### Install Vagrant plugins

I have a feeling I'm going to need this https://github.com/nsidc/vagrant-vsphere.

```bash
lawrence@cacti:~$ vagrant plugin install vagrant-vsphere
Installing the 'vagrant-vsphere' plugin. This can take a few minutes...
Fetching: mini_portile2-2.4.0.gem (100%)
Fetching: nokogiri-1.10.9.gem (100%)
Building native extensions.  This could take a while...
Fetching: trollop-2.9.10.gem (100%)
!    The 'trollop' gem has been deprecated and has been replaced by 'optimist'.
!    See: https://rubygems.org/gems/optimist
!    And: https://github.com/ManageIQ/optimist
Fetching: rbvmomi-1.13.0.gem (100%)
Fetching: vagrant-vsphere-1.13.4.gem (100%)
Installed the plugin 'vagrant-vsphere (1.13.4)'!
```

And maybe these:

```bash
lawrence@cacti:~$ vagrant plugin install vagrant-vmware-esxi
Installing the 'vagrant-vmware-esxi' plugin. This can take a few minutes...
Fetching: iniparse-1.5.0.gem (100%)
Fetching: vagrant-vmware-esxi-2.5.0.gem (100%)
Installed the plugin 'vagrant-vmware-esxi (2.5.0)'!

lawrence@cacti:~$ vagrant plugin install vagrant-reload
Installing the 'vagrant-reload' plugin. This can take a few minutes...
Fetching: vagrant-reload-0.0.1.gem (100%)
Installed the plugin 'vagrant-reload (0.0.1)'!
```
