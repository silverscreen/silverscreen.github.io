---
layout: post
title: Managing VMware with Ansible
description: Managing VMware with Ansible
summary: Managing VMware with Ansible
comments: false
---

This is a small introduction to how you can use Ansible to manage your VMware hosts.

### Install Ansible

```
lawrence@cacti:~$ sudo apt-add-repository --yes --update ppa:ansible/ansible
lawrence@cacti:~$ sudo apt update
lawrence@cacti:~$ sudo apt install ansible
```

You can test Ansible is working with the following:

```
lawrence@cacti:~$ ansible all -i "localhost," -m debug -a "msg='Hello Ansible'"
localhost | SUCCESS => {
    "msg": "Hello Ansible"
}
```

<br/>

### Setup Ansible

The default inventory file resides in `/etc/ansible/hosts`. A custom inventory path can be provided using the `-i` parameter when running commands and playbooks.

```
lawrence@cacti:~$ cat /etc/ansible/hosts 

[all:vars]
ansible_python_interpreter=/usr/bin/python3

[vcsa]
vcsa01 ansible_host=10.220.3.236

[esxi]
esxi01 ansible_host=10.220.2.122
```

You can check what's in inventory file with `ansible-inventory`:

```
lawrence@cacti:~$ ansible-inventory --list y
{
    "_meta": {
        "hostvars": {
            "esxi01": {
                "ansible_host": "10.220.2.122", 
                "ansible_python_interpreter": "/usr/bin/python3"
            }, 
            "vcsa01": {
                "ansible_host": "10.220.3.236", 
                "ansible_python_interpreter": "/usr/bin/python3"
            }
        }
    }, 
    "all": {
        "children": [
            "esxi", 
            "ungrouped", 
            "vcsa"
        ]
    }, 
    "esxi": {
        "hosts": [
            "esxi01"
        ]
    }, 
    "vcsa": {
        "hosts": [
            "vcsa01"
        ]
    }
}
```

<br/>

### Testing Connection 

After setting up the inventory file to include your servers, it’s time to check if Ansible is able to connect to these servers and run commands via SSH.

You can use the `-u` argument to specify the remote system user. When not provided, Ansible will try to connect as your current system user on the control node.

You can ping a host by doing the following: 

```
lawrence@cacti:~$ ansible esxi01 -m ping -u root
esxi01 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

<br/>

### Add A Local ESXi User

You'll notice that I'm using `-u root` because currently I have my keys added to the root account and haven't yet create my own. So let's fix that and create a local ESXi User.

Currently these are the local accounts that exist on the machine:

```
[root@cactus-uk-far-mon:~] esxcli system account list
User ID  Description
-------  -------------------------------------------
root     Administrator
dcui     DCUI User
vpxuser  VMware VirtualCenter administration account
```

I can add a new account with `esxcli system account add`, the syntax is as follows:

```
[root@cactus-uk-far-mon:~] esxcli system account add --help
Usage: esxcli system account add [cmd options]

Description: 
  add                   Create a new local user account.

Cmd options:
  -d|--description=<str>
                        User description, e.g. full name.
  -i|--id=<str>         User ID, e.g. "administrator". (required)
  -p|--password=<str>   User password. (secret)
                        WARNING: Providing secret values on the command line is insecure because it may be logged
                        or preserved in history files. Instead, specify this option with no value on the command
                        line, and enter the value on the supplied prompt.
  -c|--password-confirmation=<str>
                        Password confirmation. Required if password is specified. (secret)
                        WARNING: Providing secret values on the command line is insecure because it may be logged
                        or preserved in history files. Instead, specify this option with no value on the command
                        line, and enter the value on the supplied prompt.
```

The parameters `-p` and `-c` is for password and confirmation, both are required:

```
[root@cactus-uk-far-mon:~] esxcli system account add -i lawrence -d administrator -p -c
Enter value for 'password': 
Enter value for 'password-confirmation': 

[root@cactus-uk-far-mon:~] esxcli system account list
User ID   Description
--------  -------------------------------------------
root      Administrator
dcui      DCUI User
vpxuser   VMware VirtualCenter administration account
lawrence  administrator
```

According to VMWare the public key has to be copied to `/etc/ssh/keys-<username>/authorized_keys`.

* https://kb.vmware.com/s/article/1002866

So I'll need to copy my public key for my newly created user:

```
[root@cactus-uk-far-mon:~] cd /etc/ssh/
[root@cactus-uk-far-mon:/etc/ssh] mkdir keys-lawrence
[root@cactus-uk-far-mon:/etc/ssh] vi keys-lawrence/authorized_keys
```

Then **reload the service** with `/etc/init.d/SSH restart`:

```
[root@cactus-uk-far-mon:~] /etc/init.d/SSH restart
SSH login disabled
SSH login enabled
```

So now...

```
lawrence@cacti:~$ ssh lawrence@10.220.2.122
Connection closed by 10.220.2.122 port 22
``` 

Yup it's failing. So it turns out we have to set the permissions for my user: 

```
[root@cactus-uk-far-mon:~] esxcli system permission list
Principal  Is Group  Role   Role Description
---------  --------  -----  ------------------
dcui          false  Admin  Full access rights
root          false  Admin  Full access rights
vpxuser       false  Admin  Full access rights
```

So set the permission as `-r Admin`:

```
[root@cactus-uk-far-mon:~] esxcli system permission set -i lawrence -r Admin
[root@cactus-uk-far-mon:~] esxcli system permission list
Principal  Is Group  Role   Role Description
---------  --------  -----  ------------------
dcui          false  Admin  Full access rights
lawrence      false  Admin  Full access rights
root          false  Admin  Full access rights
vpxuser       false  Admin  Full access rights
```

And **now** I should have the relevant permissions to SSH to my ESXi host:

```
lawrence@cacti:~$ ssh lawrence@10.220.2.122
The time and date of this login have been sent to the system logs.

WARNING:
   All commands run on the ESXi shell are logged and may be included in
   support bundles. Do not provide passwords directly on the command line.
   Most tools can prompt for secrets or accept them from standard input.

VMware offers supported, powerful system administration tools.  Please
see www.vmware.com/go/sysadmintools for details.

The ESXi Shell can be disabled by an administrative user. See the
vSphere Security documentation for more information.
[lawrence@cactus-uk-far-mon:~] 
```

Phew...

<br/>

### Testing Connection (round 2)

Now that I have an account on both my VCSA and ESXi hosts, I should be able to successfully ping all my hosts within my ansible inventory: 

```
lawrence@cacti:~$ ansible all -m ping
esxi01 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
vcsa01 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

Wonderful =)

<br/>

### Installing SSL Certificates

All vCenter and ESXi servers require SSL encryption on all connections to enforce secure communication. You must enable SSL encryption for Ansible by installing the server’s SSL certificates on your Ansible control node or delegate node.

### Downloading root certificates

I had issues generating a CSR for vCenter so instead I opted to downloading the root certificates provided by VMware by navigating to the base URL of the vCenter Server system or the vCenter Server Virtual Appliance without appending port numbers or 'vsphere-client' extension.

In my case this was: `https://uk-far-vcsa-01.corp.cactus.com`. Then do `wget https://vcenter.domain.com/certs/download.zip`:

```
lawrence@cacti:~/ansible/vmware$ wget https://uk-far-vcsa-01.corp.cactus.com/certs/download.zip -P vcenter_root/
--2020-06-03 19:44:58--  https://uk-far-vcsa-01.corp.cactus.com/certs/download.zip
Resolving uk-far-vcsa-01.corp.cactus.com (uk-far-vcsa-01.corp.cactus.com)... 10.220.3.236
Connecting to uk-far-vcsa-01.corp.cactus.com (uk-far-vcsa-01.corp.cactus.com)|10.220.3.236|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8878 (8.7K) [zip]
Saving to: ‘vcenter_root/download.zip’

download.zip                           100%[==========================================================================>]   8.67K  --.-KB/s    in 0s      

2020-06-03 19:44:58 (331 MB/s) - ‘vcenter_root/download.zip’ saved [8878/8878]
```

**Alternatively** you can click **Download trusted root CA certificates** link at the bottom of the grey box on the right and download the file.

https://kb.vmware.com/s/article/2108294

<br/>

### Installing the root certificates (Debian)

Unzip the contents of `download.zip`:

```
lawrence@cacti:~/ansible/vmware/vcenter_root$ unzip download.zip 
Archive:  download.zip
  inflating: certs/lin/2b941525.r0   
  inflating: certs/mac/2b941525.r0   
  inflating: certs/win/2b941525.r0.crl  
  inflating: certs/lin/af0192ea.0    
  inflating: certs/mac/af0192ea.0    
  inflating: certs/win/af0192ea.0.crt  
  inflating: certs/lin/2b941525.0    
  inflating: certs/mac/2b941525.0    
  inflating: certs/win/2b941525.0.crt  
```

Within are three folders, each pertaining to an OS, it is important to note that the contents of these files are the same regardless of extension and OS. We are only interested in those without a `.r0*` extension:

```
root@cacti:~/ansible/vmware/vcenter_root/certs/lin# ls *.0
2b941525.0  af0192ea.0
```

Now create a folder for our VCSA certs with `/usr/local/share/ca-certificates`:

```
root@cacti:~/ansible/vmware/vcenter_root/certs/lin# mkdir /usr/local/share/ca-certificates/vcsa
```

And move the root certificates to our newly created folder whilst also giving them the file extension `.crt`:

```
root@cacti:~/ansible/vmware/vcenter_root/certs/lin# mv 2b941525.0 /usr/local/share/ca-certificates/vcsa/2b941525.crt
root@cacti:~/ansible/vmware/vcenter_root/certs/lin# mv af0192ea.0 /usr/local/share/ca-certificates/vcsa/af0192ea.crt
```

Ensure they have the correct permissions:

```
root@cacti:~/ansible/vmware/vcenter_root/certs/lin# chmod 644 /usr/local/share/ca-certificates/vcsa/*
root@cacti:~/ansible/vmware/vcenter_root/certs/lin# chmod 755 /usr/local/share/ca-certificates/vcsa
```

Lastly update the CA certificates directory `/etc/ssl/certs`:

```
root@cacti:/usr/local/share/ca-certificates/vcsa# sudo update-ca-certificates 
Updating certificates in /etc/ssl/certs...
rehash: warning: skipping duplicate certificate in ca.pem
2 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

Now that's done, I can test that my certificates were installed correctly and are working by using `openssl s_client` to connect to the vCenter server:

```
lawrence@cacti:/usr/local/share/ca-certificates/vcsa$ openssl s_client -connect uk-far-vcsa-01.corp.cactus.com:443 -CApath /etc/ssl/certs/
CONNECTED(00000005)
depth=1 CN = CA, DC = vsphere, DC = local, C = US, ST = California, O = uk-far-vcsa-01.corp.cactus.com, OU = VMware Engineering
verify return:1
depth=0 CN = uk-far-vcsa-01.corp.cactus.com, C = US
verify return:1
---
Certificate chain
 0 s:CN = uk-far-vcsa-01.corp.cactus.com, C = US
   i:CN = CA, DC = vsphere, DC = local, C = US, ST = California, O = uk-far-vcsa-01.corp.cactus.com, OU = VMware Engineering
---
Server certificate
-----BEGIN CERTIFICATE-----
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

BGv2
-----END CERTIFICATE-----
subject=CN = uk-far-vcsa-01.corp.cactus.com, C = US

issuer=CN = CA, DC = vsphere, DC = local, C = US, ST = California, O = uk-far-vcsa-01.corp.cactus.com, OU = VMware Engineering

---
No client certificate CA names sent
Peer signing digest: SHA512
Peer signature type: RSA
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 1534 bytes and written 455 bytes
Verification: OK
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: 
    Session-ID-ctx: 
    Master-Key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1591207474
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
---
```

As you can see from the output above, the connection is successful, and now I am ready to begin working with my vCenter host in Ansible.

<br/>

### Changing the name of an ESX or ESXi host (1010821)

https://kb.vmware.com/s/article/1010821

1. Remove the ESXi host from the VCSA inventory.
2. https://blogs.vmware.com/vsphere/2019/08/changing-your-vcenter-servers-fqdn.html
3. Login to VCSA via `https://10.220.3.236:5480`

```
Hostname:
10.220.3.236
```

4. Go to summary `https://10.220.3.236:5480/ui/summary`.
5. Click on **Networking** to continue.
6. Network interfaces are displayed on the right under Network Settings. Click Edit continue.


```
lawrence@cacti:~/vmware/10.220.2.122$ ssh 10.220.3.236 -l root

VMware vCenter Server Appliance 6.7.0.10000

Type: vCenter Server with an embedded Platform Services Controller

Last login: Fri May 29 10:06:50 2020 from 10.220.101.104
root@uk-far-vcsa-01 [ ~ ]# /opt/vmware/share/vami/vami_config_net

 Main Menu 

0)      Show Current Configuration (scroll with Shift-PgUp/PgDown)
1)      Exit this program
2)      Default Gateway
3)      Hostname
4)      DNS
5)      Proxy Server
6)      IP Address Allocation for eth0
Enter a menu number [0]: 3

Warning: if any of the interfaces for this VM use DHCP,
the Hostname, DNS, and Gateway parameters will be
overwritten by information from the DHCP server.

Type Ctrl-C to go back to the Main Menu

New hostname [uk-far-vcsa-01.corp.cactus.com]: uk-far-vcsa-01.corp.cactus.com
== set_ipv4 ==
DEFULT_INT: eth0
DEFAULT_IPV4: 10.220.3.236
HN: uk-far-vcsa-01
DN: corp.cactus.com
==============

== set_ipv6 ==
DEFULT_INT: eth0
DEFAULT_IPV6: 
HN: uk-far-vcsa-01
DN: corp.cactus.com
==============

Host name has been set to uk-far-vcsa-01.corp.cactus.com
```

#### Now reboot the vCenter Server Appliance:

1. In the vCenter Server Appliance Management Interface, click Summary.
2. From the top menu pane, click the Actions drop-down menu.
3. Click Reboot or Shutdown to restart or power off the virtual machine.
4. In the confirmation dialog box, click Yes to confirm the operation. 


----- 

### VMware Prerequisites

So now I should have the following Prerequisites met:

* SSH enabled on my ESXi host
* ESXi SSL certificates installed for Ansible

<br/>

#### Using the VMware dynamic inventory plugin

The preferred way (according to the Ansible documentation - link below) to manage your hosts is to use the **VMware dynamic inventory plugin**, which dynamically queries VMware APIs and notifies Ansible which nodes can be managed.

* https://docs.ansible.com/ansible/latest/scenario_guides/vmware_scenarios/vmware_inventory.html

The `pyVmomi` python SDK for VMware vSphere API must be installed on the Ansible node.

* https://github.com/vmware/pyvmomi

In this case I have already installed `pyvmomi`:

```
lawrence@cacti:~/vmware$ sudo pip3 install pyvmomi
The directory '/home/lawrence/.cache/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
The directory '/home/lawrence/.cache/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
Requirement already satisfied: pyvmomi in /usr/local/lib/python3.6/dist-packages
Requirement already satisfied: requests>=2.3.0 in /usr/lib/python3/dist-packages (from pyvmomi)
Requirement already satisfied: six>=1.7.3 in /home/lawrence/.local/lib/python3.6/site-packages (from pyvmomi)
```

#### Enable the inventory plugin

Most inventory plugins shipped with Ansible are disabled by default and need to be whitelisted in your `ansible.cfg` file in order to function.

To use the `pyvmomi` plugin, we need to enable it first by specifying the following:

```
lawrence@cacti:/etc/ansible$ ls
ansible.cfg  hosts  roles
lawrence@cacti:/etc/ansible$ sudo vim ansible.cfg 

[inventory]
enable_plugins = vmware_vm_inventory
```

Now we need to create a file that ends in `.vmware.yml` or `.vmware.yaml` in our working directory:

```
lawrence@cacti:~/ansible/vmware$ vim vmwareinvt.vmware.yml

plugin: vmware_vm_inventory
strict: False
hostname: 10.220.2.122
username: lawrence
password: mypassword
validate_certs: False
with_tags: True
```

Executing `ansible-inventory --list -i vmwareinvt.vmware.yml` will create a list of VMware instances that are ready to be configured using Ansible.

<br/>

### Using vaulted configuration files

As our inventory configuration file `vmwareinvt.vmware.yml` contains our password in plain text, it would be a good idea to encrypt the entire inventory configuration file using `ansible-vault`:

```
lawrence@cacti:~/ansible/vmware$ ansible-vault encrypt vmwareinvt.vmware.yml
New Vault password: 
Confirm New Vault password: 
Encryption successful
```

Now our inventory file is encrypted:

```
lawrence@cacti:~/ansible/vmware$ cat vmwareinvt.vmware.yml 
$ANSIBLE_VAULT;1.1;AES256
37643931666230363338376639393666666139346236623430653631373338653463366637666335
3232623162396438653432633132393063626565663238320a393734343838373736346220363832
35653832653035353361613533316530336432363739643333626535653839363066386331313132
6262363034633037630a353534326261363061363063303866303030636535643066316533333338
63353838616433343838336337386439626430326361393330623737353062333132643762353332
...
```

It can be viewed with `ansible-vault view` after which it will ask you for the vault password:

```
lawrence@cacti:~/ansible/vmware$ ansible-vault view vmwareinvt.vmware.yml 
Vault password: 
plugin: vmware_vm_inventory
strict: False
hostname: 10.220.2.122
username: lawrence
password: mypassword
validate_certs: False
with_tags: True
```

And now if we were to use the file we would do the following:

```
$ ansible-inventory -i filename.vmware.yml --list --vault-password-file=/path/to/vault_password_file
```

More information on Ansible Vault here https://docs.ansible.com/ansible/latest/user_guide/vault.html

<br/>

### Providing vault passwords

When all data is encrypted using a single password the --ask-vault-pass or --vault-password-file cli options should be used.

For example, to use a password store in the text file `/path/to/my/vault-password-file`:

```
ansible-playbook --vault-password-file /path/to/my/vault-password-file site.yml
```

To prompt for a password:

```
ansible-playbook --ask-vault-pass site.yml
```

To get the password from a vault password executable script my-vault-password.py:

```
ansible-playbook --vault-password-file my-vault-password.py
```

Here I am able to call my vault password file with:

```
lawrence@cacti:~/ansible/vmware$ ansible-vault view vmwareinvt.vmware.yml --vault-password-file vaultpasswd 
plugin: vmware_vm_inventory
strict: False
hostname: 10.220.2.122
username: lawrence
password: mypassword
validate_certs: False
with_tags: True
```

#### Permanently decrypting files

The file can be **permanently decrypted** with the following:

```
lawrence@cacti:~/ansible/vmware$ ansible-vault decrypt vmwareinvt.vmware.yml --vault-password-file vaultpasswd 
Decryption successful
```

<br/>


### VMware Dynamic Inventory Plugin

So I ran into an issue with the vSphere automation SDK which I solved by running it within a **virtual environment** (see my post on virtual environments):

```
git clone https://github.com/vmware/vsphere-automation-sdk-python.git
cd vsphere-automation-sdk-python

pip3 install --upgrade --force-^Cinstall -r requirements.txt --extra-index-url file://$PWD/lib
```

So now I have my dynamic inventory which uses `pyvmomi` and the vSphere Automation SDK, which supports REST API features like tagging etc:

```
(ansible-vmware) lawrence@cacti:~/ansible/vmware$ cat vmwareinvt.vmware.yml 
plugin: vmware_vm_inventory
strict: False
hostname: uk-far-vcsa-01.corp.cactus.com
username: lawrencelong
password: mypassword
validate_certs: False
with_tags: True
properties:
- 'name'
- 'guest.ipAddress'
```

And now I can execute my inventory, creating a list of VMware instances that are ready to be configured with Ansible:

```
(ansible-vmware) lawrence@cacti:~/ansible/vmware$ ansible-inventory -i vmwareinvt.vmware.yml --list 
{
    "_meta": {
        "hostvars": {
            "DLE_564df4bf-883c-b081-3b19-63d0b6cb473f": {
                "guest.ipAddress": null,
                "name": "DLE"
            },
            "Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "Junos Space"
            },
            "Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "Log Collector"
            },
            "VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "VIRL"
            },
            "gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "gns3_vm"
            },
            "ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "ubuntu_chronos"
            },
            "ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "ubuntu_gns3"
            },
            "uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX": {
                "ansible_host": "10.220.3.100",
                "guest.ipAddress": "10.220.3.100",
                "name": "uk-far-ca-01.corp.cactus.com"
            },
            "uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX": {
                "ansible_host": "10.220.3.236",
                "guest.ipAddress": "10.220.3.236",
                "name": "uk-far-vcsa-01.corp.cactus.com"
            }
        }
    },
    "all": {
        "children": [
            "centos7_64Guest",
            "other3xLinux64Guest",
            "otherLinux64Guest",
            "poweredOff",
            "poweredOn",
            "rhel6_64Guest",
            "ubuntu64Guest",
            "ungrouped"
        ]
    },
    "centos7_64Guest": {
        "hosts": [
            "DLE_564df4bf-883c-b081-3b19-XXXXXXXXX"
        ]
    },
    "other3xLinux64Guest": {
        "hosts": [
            "uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX"
        ]
    },
    "otherLinux64Guest": {
        "hosts": [
            "Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX"
        ]
    },
    "poweredOff": {
        "hosts": [
            "DLE_564df4bf-883c-b081-3b19-XXXXXXXXX",
            "Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX",
            "Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX",
            "VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX",
            "gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX",
            "ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX",
            "ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX"
        ]
    },
    "poweredOn": {
        "hosts": [
            "uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX",
            "uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX"
        ]
    },
    "rhel6_64Guest": {
        "hosts": [
            "Junos Space_564d8401-d18b-80ee-5be2-c73ffa715f79"
        ]
    },
    "ubuntu64Guest": {
        "hosts": [
            "VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX",
            "gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX",
            "ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX",
            "ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX",
            "uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX"
        ]
    }
}
```

Pretty neat huh!

<br/>
 

### Managing your hosts 

I have the following inventory file: 

```
(ansible-vmware) lawrence@cacti:~/ansible/vmware$ cat vmwareinvt.vmware.yml 
plugin: vmware_vm_inventory
strict: False
hostname: uk-far-vcsa-01.corp.cactus.com
username: lawrencelong
password: mypassword
esxi_hostname: uk-far-esxi-02.corp.cactus.com
validate_certs: False
with_tags: True
properties:
- 'name'
- 'guest.ipAddress'
```

I will just test the configuration script with below command to check script is able to connect vCenter and fetch the vms list correctly. 

After running below command it was showing VMs list with there configuration successfully:

```
(ansible-vmware) lawrence@cacti:~/ansible/vmware$ ansible-inventory --list -i vmwareinvt.vmware.yml 
{
    "_meta": {
        "hostvars": {
            "DLE_564df4bf-883c-b081-3b19-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "DLE"
            },
            "Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "Junos Space"
            },
            "Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "Log Collector"
            },
            "VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "VIRL"
            },
            "gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "gns3_vm"
            },
            "ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "ubuntu_chronos"
            },
            "ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX": {
                "guest.ipAddress": null,
                "name": "ubuntu_gns3"
            },
            "uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX": {
                "ansible_host": "10.220.3.100",
                "guest.ipAddress": "10.220.3.100",
                "name": "uk-far-ca-01.corp.cactus.com"
            },
            "uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX": {
                "ansible_host": "10.220.3.236",
                "guest.ipAddress": "10.220.3.236",
                "name": "uk-far-vcsa-01.corp.cactus.com"
            }
        }
    },
    "all": {
        "children": [
            "centos7_64Guest",
            "other3xLinux64Guest",
            "otherLinux64Guest",
            "poweredOff",
            "poweredOn",
            "rhel6_64Guest",
            "ubuntu64Guest",
            "ungrouped"
        ]
    },
    "centos7_64Guest": {
        "hosts": [
            "DLE_564df4bf-883c-b081-3b19-XXXXXXXXX"
        ]
    },
    "other3xLinux64Guest": {
        "hosts": [
            "uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX"
        ]
    },
    "otherLinux64Guest": {
        "hosts": [
            "Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX"
        ]
    },
    "poweredOff": {
        "hosts": [
            "DLE_564df4bf-883c-b081-3b19-XXXXXXXXX",
            "Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX",
            "Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX",
            "VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX",
            "gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX",
            "ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX",
            "ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX"
        ]
    },
    "poweredOn": {
        "hosts": [
            "uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX",
            "uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX"
        ]
    },
    "rhel6_64Guest": {
        "hosts": [
            "Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX"
        ]
    },
    "ubuntu64Guest": {
        "hosts": [
            "VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX",
            "gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX",
            "ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX",
            "ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX",
            "uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX"
        ]
    }
}
```

Next I tested another command to retrieve the list in graph form:

```
(ansible-vmware) lawrence@cacti:~/ansible/vmware$ ansible-inventory --graph -i vmwareinvt.vmware.yml 
@all:
  |--@centos7_64Guest:
  |  |--DLE_564df4bf-883c-b081-3b19-XXXXXXXXX
  |--@other3xLinux64Guest:
  |  |--uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX
  |--@otherLinux64Guest:
  |  |--Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX
  |--@poweredOff:
  |  |--DLE_564df4bf-883c-b081-3b19-XXXXXXXXX
  |  |--Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX
  |  |--Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX
  |  |--VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX
  |  |--gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX
  |  |--ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX
  |  |--ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX
  |--@poweredOn:
  |  |--uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX
  |  |--uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX
  |--@rhel6_64Guest:
  |  |--Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX
  |--@ubuntu64Guest:
  |  |--VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX
  |  |--gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX
  |  |--ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX
  |  |--ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX
  |  |--uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX
  |--@ungrouped:
```  

As you can see the Information gathering was successful.

Above all the commands where one-liner ad-hoc. I want to use this plugin in the playbook. Below is the playbook I am using, once it is connected to vCenter successfully, all the information is gathered as facts into global default variable vars. And once information is gathered I can use the same variable to process further for automation: 

```
(ansible-vmware) lawrence@cacti:~/ansible/vmware$ cat vcloudexample.yml 
---
  - name: Get vms list
    hosts: localhost
    gather_facts: no

    tasks:
      #- name: gather list
        #set_fact:
          #vmlist: "{{ lookup('env','vcenter_host') }}"

      - debug:
          var: vars.groups.all
```

```
(ansible-vmware) lawrence@cacti:~/ansible/vmware$ ansible-playbook vcloudexample.yml -i vmwareinvt.vmware.yml 

PLAY [Get vms list] **************************************************************************************************************************************

TASK [debug] *********************************************************************************************************************************************
ok: [localhost] => {
    "vars.groups.all": [
        "DLE_564df4bf-883c-b081-3b19-XXXXXXXXX",
        "ubuntu_chronos_564df294-bfa3-0f1b-696e-XXXXXXXXX",
        "Log Collector_564daa9a-7fb6-dde7-f269-XXXXXXXXX",
        "gns3_vm_564d12f0-0111-999e-2451-XXXXXXXXX",
        "Junos Space_564d8401-d18b-80ee-5be2-XXXXXXXXX",
        "ubuntu_gns3_564d0c5a-5c01-5636-6d3b-XXXXXXXXX",
        "VIRL_564d11e6-e98f-ff26-15cb-XXXXXXXXX",
        "uk-far-ca-01.corp.cactus.com_564d6745-fc40-d707-d7af-XXXXXXXXX",
        "uk-far-vcsa-01.corp.cactus.com_564d92ed-ea58-3c15-6853-XXXXXXXXX"
    ]
}

PLAY RECAP ***********************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```


---- 

### VMware guest info

More information about the `vmware_guest_info` Ansible module here:

https://docs.ansible.com/ansible/latest/modules/vmware_guest_info_module.html#vmware-guest-info-module

So this is my playbook, here I'm able to give a `uuid` instead of a `hostname` for the VM in question: 

```
---
- name: Create a VM from a template
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Gather info from standalone ESXi server
      vmware_guest_info:
        hostname: uk-far-vcsa-01.corp.cactus.com
        username: lawrencelong
        password: mypassword
        datacenter: cactusDC
        #esxi_hostname: uk-far-esxi-02.corp.cactus.com
        validate_certs: yes
        uuid: 564d6745-fc40-d707-d7af-XXXXXXX
      register: guest_facts
      delegate_to: localhost

    - name: display guest facts
      debug: var=guest_facts
```

Output:

```
TASK [display guest facts] ******************************************************************************************************************************
ok: [localhost] => {
    "guest_facts": {
        "changed": false,
        "failed": false,
        "instance": {
            "annotation": "",
            "current_snapshot": null,
            "customvalues": {},
            "guest_consolidation_needed": false,
            "guest_question": null,
            "guest_tools_status": "guestToolsRunning",
            "guest_tools_version": "11269",
            "hw_cluster": null,
            "hw_cores_per_socket": 1,
            "hw_datastores": [
                "220122_LOCAL"
            ],
            "hw_esxi_host": "uk-far-esxi-02.corp.cactus.com",
            "hw_eth0": {
                "addresstype": "generated",
                "ipaddresses": [
                    "10.220.3.100",
                    "fe80::20c:29ff:fe06:b43f"
                ],
                "label": "Network adapter 1",
                "macaddress": "XX.XX.XX.XX.XX.XX",
                "macaddress_dash": "XX.XX.XX.XX.XX.XX",
                "portgroup_key": null,
                "portgroup_portkey": null,
                "summary": "Server Network"
            },
            "hw_files": [
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.vmx",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.nvram",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.vmsd",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/vmware-2.log",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/vmware-1.log",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/vmware.log",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.vmdk"
            ],
            "hw_folder": "/cactusDC/vm",
            "hw_guest_full_name": "Ubuntu Linux (64-bit)",
            "hw_guest_ha_state": null,
            "hw_guest_id": "ubuntu64Guest",
            "hw_interfaces": [
                "eth0"
            ],
            "hw_is_template": false,
            "hw_memtotal_mb": 1024,
            "hw_name": "uk-far-ca-01.corp.cactus.com",
            "hw_power_status": "poweredOn",
            "hw_processor_count": 1,
            "hw_product_uuid": "564d6745-fc40-d707-d7af-XXXXXXX",
            "hw_version": "vmx-14",
            "instance_uuid": "521a4352-6ffd-8132-a667-XXXXXXX",
            "ipv4": "10.220.3.100",
            "ipv6": null,
            "module_hw": true,
            "moid": "vm-29",
            "snapshots": [],
            "vimref": "vim.VirtualMachine:vm-29",
            "vnc": {}
        }
    }
} 
```

And as you can see there is a lot of very cool information that you can begin working with.

<br/>

### Working with Return Values

Ansible modules normally return a data structure that can be registered into a variable, or seen directly when output by the ansible program. Each module can optionally document its own unique return values (visible through ansible-doc and on the main docsite).

https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html#common-return-values
https://docs.ansible.com/ansible/latest/modules/vmware_guest_info_module.html#vmware-guest-info-module

New playbook that gathers information using the `vmware_guest_info` module from a VM by either specifying the `uuid` or the `name`:

```yaml
---
- name: Gather information from VMs
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Gather some info from a guest using the vSphere API output schema
      vmware_guest_info:
        hostname: uk-far-vcsa-01.corp.cactus.com
        username: lawrencelong
        password: mypassword
        datacenter: cactusDC
        #esxi_hostname: uk-far-esxi-02.corp.cactus.com
        validate_certs: yes
        uuid: 564d6745-fc40-d707-d7af-XXXXXXXXX
        #name: uk-far-ca-01.corp.cactus.com
        #schema: "vsphere"
        #properties: ["config.hardware.memoryMB", "guest.disk", "overallStatus"]
      register: guest_facts
      delegate_to: localhost

    - debug:
        var: guest_facts

    - assert:
        that:
          - "guest_facts['instance']['hw_name'] is defined"

    - debug:
        msg: "HW_NAME is {{ guest_facts['instance']['hw_name'] }}"
```

The resulting output is returned within a dictionary: 

```bash
TASK [debug] ********************************************************************************************************************************************
ok: [localhost] => {
    "guest_facts": {
        "changed": false,
        "failed": false,
        "instance": {
            "annotation": "",
            "current_snapshot": null,
            "customvalues": {},
            "guest_consolidation_needed": false,
            "guest_question": null,
            "guest_tools_status": "guestToolsRunning",
            "guest_tools_version": "11269",
            "hw_cluster": null,
            "hw_cores_per_socket": 1,
            "hw_datastores": [
                "220122_LOCAL"
            ],
            "hw_esxi_host": "uk-far-esxi-02.corp.cactus.com",
            "hw_eth0": {
                "addresstype": "generated",
                "ipaddresses": [
                    "10.220.3.100",
                ],
                "label": "Network adapter 1",
                "macaddress": "XX.XX.XX.XX.XX.XX",
                "macaddress_dash": "XX.XX.XX.XX.XX.XX",
                "portgroup_key": null,
                "portgroup_portkey": null,
                "summary": "Server Network"
            },
            "hw_files": [
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.vmx",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.nvram",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.vmsd",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/vmware-2.log",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/vmware-1.log",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/vmware.log",
                "[220122_LOCAL] uk-far-ca-01.corp.cactus.com/uk-far-ca-01.corp.cactus.com.vmdk"
            ],
            "hw_folder": "/cactusDC/vm",
            "hw_guest_full_name": "Ubuntu Linux (64-bit)",
            "hw_guest_ha_state": null,
            "hw_guest_id": "ubuntu64Guest",
            "hw_interfaces": [
                "eth0"
            ],
            "hw_is_template": false,
            "hw_memtotal_mb": 1024,
            "hw_name": "uk-far-ca-01.corp.cactus.com",
            "hw_power_status": "poweredOn",
            "hw_processor_count": 1,
            "hw_product_uuid": "564d6745-fc40-d707-d7af-XXXXXXX",
            "hw_version": "vmx-14",
            "instance_uuid": "521a4352-6ffd-8132-a667-XXXXXXX",
            "ipv4": "10.220.3.100",
            "ipv6": null,
            "module_hw": true,
            "moid": "vm-29",
            "snapshots": [],
            "vimref": "vim.VirtualMachine:vm-29",
            "vnc": {}
        }
    }
}

TASK [assert] *******************************************************************************************************************************************
ok: [localhost] => {
    "changed": false,
    "msg": "All assertions passed"
}

TASK [debug] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "HW_NAME is uk-far-ca-01.corp.cactus.com"
}
```

Within the first task I store this output by registering it as a variable `guest_facts`: 

```yaml
        register: guest_facts
```

I then call my variable with the following `debug`, with the output returned in a dictionary structure in my terminal:

```yaml
    - debug:
        var: guest_facts
```

Now I am ready to begin working with **return values**. The **key** to access our dictionary, according to the modules documentation is `instance`. 

So from looking at our output, as an example, we can call the **key** `hw_name`:

```yaml
            "hw_is_template": false,
            "hw_memtotal_mb": 1024,
            "hw_name": "uk-far-ca-01.corp.cactus.com",
```

And perform a check against it to confirm it is `defined`:

```yaml
    - assert:
        that:
          - "guest_facts['instance']['hw_name'] is defined"
```          

Output:

```bash
TASK [assert] *******************************************************************************************************************************************
ok: [localhost] => {
    "changed": false,
    "msg": "All assertions passed"
}
```

Lastly I am able to call this key by referring to it as a variable `{{ guest_facts['instance']['hw_name'] }}`: 

```yaml
    - debug:
        msg: "HW_NAME is {{ guest_facts['instance']['hw_name'] }}"
```

With result returning the **value** of that key, being `uk-far-ca-01.corp.cactus.com`:

```bash
TASK [debug] ********************************************************************************************************************************************
ok: [localhost] => {
    "msg": "HW_NAME is uk-far-ca-01.corp.cactus.com"
}
```

Now you know how to work with return values, you can start incorporating them into your playbooks =)

----

### Conclusion

So there you have it. Hopefully this has helped you understand how you too can begin managing your VMware suite with Ansible. 

There are a ton of modules at your disposal, and using what I learnt I was able to then construct playbooks that now perform roles like **backups**, **replication between datastores**, and **virtual machine deployments**. 