---
layout: post
title: Collecting RSI and System Log Files in Junos
description: Collecting RSI and System Log Files in Junos
summary: Collecting RSI and System Log Files in Junos
comments: false
---

I recently had to contact JTAC and submit some logs to them. I had not done this before so thought I would document the process of how to request and submit support information. 

In this example I will be peforming this on two Juniper SRX firewalls configured as a Chassis Cluster.

### Issue the `request support information` command

#### `node0`:

```
lawrence@CACTUS-UK-FAR-SRX01> request support information | save /var/log/5-02-19-node0.txt 
Wrote 48609 lines of output to '/var/log/5-02-19-node0.txt'
```

Then move to `node1` by doing the following:

```
lawrence@CACTUS-UK-FAR-SRX01> request routing-engine login node 1 
```

#### `node1`:

```
--- JUNOS 18.4R1.8 built 2018-12-17 03:29:17 UTC
{secondary:node1}
lawrence@CACTUS-UK-FAR-SRX02> request support information | save /var/log/5-02-19-node1.txt
Wrote 48620 lines of output to '/var/log/5-02-19-node1.txt'
```

<br/>

### Compress the logs with `compress source`

#### `node0`:

```
{primary:node0}
file archive compress source /var/log/* destination /var/tmp/5-02-19-node0-logs.tgz
```

#### `node1`:

```
{secondary:node1}
file archive compress source /var/log/* destination /var/tmp/5-02-19-node1-logs.tgz
```

To ensure the `/var/log/` directory was properly archived, you can check the filesize using the commands:

```
file list /var/tmp/CURRENT-DATE.tgz detail
```

Then use `file copy` to copy the files from one node in the chassis to the other:

#### `node1`:

```
{secondary:node1}
lawrence@CACTUS-UK-FAR-SRX02> file copy /var/tmp/5-02-19-node1-logs.tgz node0:/var/tmp/    
```

<br/>

### Lastly transfer them off the device

#### `node0`:

```
{primary:node0}
lawrence@CACTUS-UK-FAR-SRX01> scp /var/tmp/5-02-19-node0-logs.tgz lawrence@192.168.17.190:.
lawrence@CACTUS-UK-FAR-SRX01> scp /var/tmp/5-02-19-node1-logs.tgz lawrence@192.168.17.190:.
```

<br/>

---- 

### Collect Logs from an EX-SW Virtual Chassis 

Upon logging into a Virtual Chassis you should arrive at the master switch `master:0`:

```
{master:0}
lawrence@CACTUS-UK-FAR-SW01>
```

<br/>

### Drop to `shell` and compress `/var/log`

Start the shell with `start shell` and use the `tar` command to compress `/var/log` and its contents to the `/var/tmp` directory: 

```
root% tar -zcvf /var/tmp/varlog-mem0.tar.gz /var/log/*
```

<br/>

### Confirm the logs were successfully compressed

```
root% ls /var/tmp
.ssh varlog-mem0.tar.gz
```

### Exit and request the next member in the chassis

Then `exit` from the shell.

```
{master:0}
root% exit
```

And request the next member of the virtual chassis with its **member id**. You can verify the member you are connected to with `show virtual chassis` and the member you are on will be denoted with an asterisk `*`.

```
root> request session member 1
```

<br/>

### Now do the same as the previous steps but this time denoting the member in the file name

Repeat the steps with each member id until all the member logs are loaded to `master:0`, name each file according the member id to easily identify.

```
root% tar -zcvf /var/tmp/varlog-mem1.tar.gz /var/log/*
```

<br/>

### Copy the files to master switch `member:0`

```
root% cli
```

```
root> file copy fpc1:/var/tmp/varlog-mem1.tar.gz fpc0:/var/tmp/
```

### Lastly transfer them off the device

```
{master:0}
lawrence@CACTUS-UK-FAR-SW01> scp /var/tmp/5-02-19-node1-logs.tgz lawrence@192.168.17.190:.
```
