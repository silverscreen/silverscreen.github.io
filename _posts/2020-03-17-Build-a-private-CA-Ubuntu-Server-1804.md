---
layout: post
title: Build a private CA Ubuntu Server 18.04
description: Build a private CA Ubuntu Server 18.04
summary: Build a private CA Ubuntu Server 18.04
comments: false
---

Recently I had the pleasure of building a another private CA that could be managed by the IT team for signing internal CSRs. I had previously already built a CA for internal use through which I signed our internal VPNs as the technology were are using prevented us from using a PSK. 

Here is a quick an simple guide to SSL certificates and how to build your own CA for private use. 

----

### How SSL works 

A CA is responsible for issuing digital certs to verify identities on the public internet. The job of a SSL cert is to initiate secure sessions with the user's browser via SSL.

So let's say you have a website on the internet, and you require a valid SSL certificate. That SSL certificate can then be validated, so that the connecting host can confirm that the website is secure and thus an encrypted session can be established.

Securing an SSL certificate is done by the following:

1. **Create CSR** (Certificate Signing Request). This generates a **private** and **public key pair** on your server. The CSR file contains your **public key**.
2. The **CA signs the CSR** using its **root cert**, creating a data structure that matches your private key. **The CA never sees your private key**.
3. A signed SSL cert is received from the CA which can be installed on that web server. 

Certs are signed with an **expiry date** and a **digital signature** that can be validated. Anyone can create a certificate, but browsers only trust certificates that come from an organization on their list of trusted CAs.

----

**Intermediate CA Certificates** An intermediate certificate can also be installed for an extra layer of security. This is because the CA does not need to issue certificates directly from the CA's root cert. Thus the trust-chain begins at the trusted root CA, through the intermediate cert, finally ending with the issued SSL cert. These are known as **chained root certificates**.

_If the Primary Root CA is not in the browser, and as such, not implicitly trusted by the browsers that verify the signature, the Intermediate CA must be installed on the server acting as a chain link between the browser root and the server certificate. The Intermediate CA is chained to RSA Secure server CA which the browser already trusts. - Source: DigiCert_

----

So now that you have a signed SSL certificate loaded on your web server, how are these SSL connections made? 

When you first make a connection to a web server secured with SSL, a **SSL handshake** is performed. The host initiating the connection will request that the server identify itself. In response, the web server serves a copy of its **SSL cert** as well as its **public key**. 

A check is then performed by the host, confirming whether or not the SSL cert can be trusted. This is done by checking whether or not the issuer of the certificate (the CA) is already trusted by the local installation's set of CA certificates. It confirms whether the SSL cert is unexpired, unrevoked, and that its common name is valid for the website. **There is no contact with the issuing CA.** 

**If the intiating host (or browser) trusts the certificate, it creates, encrypts, and sends back a symmetric session key using the web server's public key.**

The **web server decrypts the symmetric session key** using its private key and sends back an ack encrypted with the session key so that an encrypted session can be initiated.

The result is that now the connecting host and web server can encrypt all transmitted data with the **session key**.

More informatin on SSL cryptography here:

* https://www.digicert.com/ssl/
* https://www.digicert.com/ssl-cryptography.htm#:~:text=PKI%20uses%20a%20hybrid%20cryptosystem,the%20SSL%20Handshake%20is%20symmetric.


<br/>

### So why build a CA server?

If your intentions are to run encrypted session for internal applications, then instead of paying for a trusted CA like DigiCert for all your digital certificates, it can be useful to build your own CA for internal use only.

Building our own CA means we can issues certificates for users, servers, and specific services. 

For this example, I will setup a private CA on a Ubuntu server, and generate and sign a certificate for one of my VMware hosts using my CA. I will also need to import the CA server's public certificate into my operating system's certificate store so that the trust between CA and host can be validated.

<br/>

### Prerequisites

First I'll need to provision a new Ubuntu 20.04 server which I'll give the following settings:

```
uk-far-ca-01.corp.cactus.com
10.220.3.100

vCPUs: 1
Memory: 1024 MB
Capacity: 20GB
ISO: ISO/ubuntu-20.04-live-server-amd64.iso
```

This will be my **CA Server**. It will be a standalone system and will only be used to **import**, **sign**, and **revoke** certificate requests. 

----

**Heads Up!** It should not run any other services. And for security purposes, ideally it will be offline or completely shut down when not actively working on it.

----

### Securing your CA

First things first, I'm going to secure my CA with only SSH and disable passwords. I'll do this by making some edits to `sshd_config` so that SSH is controlled only by root.


By default ssh keys are read from the user's home directory `~/.ssh/authorized_keys`.

The downside of this is that the user can add new keys that authorize logins to their account, so I'm going to tighten this up so only root controls this. It's also a nice consolidated way of managing things.

All keys will be managed under a root owned directory; `/etc/ssh/keys` with a sub-dir for each user and within that an `authorized_keys` file pertaining to that user:

```
root@uk-far-ca-01:/etc/ssh# mkdir -p keys/lawrence
root@uk-far-ca-01:/etc/ssh# vim keys/lawrence/authorized_keys
```

Once I've added my `.pub` key, I can make the amendments to `sshd_config` and change the default location for `authorized_keys` as well as disable `PasswordAuthentication`:

```
root@uk-far-ca-01:/etc/ssh# vim sshd_config

#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
AuthorizedKeysFile /etc/ssh/keys/%u/authorized_keys

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no
```

```
root@uk-far-ca-01:/etc/ssh# tree keys/
keys/
└── lawrence
    └── authorized_keys
```

Finally, restart the SSH service with:

```
root@uk-far-ca-01:/etc/ssh# systemctl restart ssh.service
```

```
root@uk-far-ca-01:/etc/ssh# systemctl status ssh.service
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-06-02 14:35:39 UTC; 14s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 41297 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 42208 (sshd)
      Tasks: 1 (limit: 1075)
     Memory: 1.1M
     CGroup: /system.slice/ssh.service
             └─42208 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

Jun 02 14:35:39 uk-far-ca-01 systemd[1]: Starting OpenBSD Secure Shell server...
Jun 02 14:35:39 uk-far-ca-01 sshd[42208]: Server listening on 0.0.0.0 port 22.
Jun 02 14:35:39 uk-far-ca-01 sshd[42208]: Server listening on :: port 22.
Jun 02 14:35:39 uk-far-ca-01 systemd[1]: Started OpenBSD Secure Shell server.
```

More information on SSH here: https://help.ubuntu.com/community/SSH/OpenSSH/Keys.

### Managing the CA

Lastly I will create a `ca_group` so that the CA can be managed by additional users from my team:

```
lawrence@uk-far-ca-01:~$ sudo groupadd ca_group
```

Now I can simply add users to this group and they will the required permissions to manage the CA:

```
lawrence@uk-far-ca-01:~$ sudo adduser lawrence ca_group
Adding user `lawrence' to group `ca_group' ...
Adding user lawrence to group ca_group
Done.
```

```
root@uk-far-ca-01:/home/lawrence# cd /home/
root@uk-far-ca-01:/home# chgrp -R ca_group cactus
root@uk-far-ca-01:/home# chmod -R g+rwx cactus
```

```
root@uk-far-ca-01:/home# ls -al cactus/
total 52
drwxrwxr-x 5 cactus  ca_group  4096 Jun  1 15:59 .
drwxr-xr-x 4 root root      4096 Jun  1 12:53 ..
-rw-rwx--- 1 cactus  ca_group  1912 Jun  2 15:11 .bash_history
-rw-rwxr-- 1 cactus  ca_group   220 Jun  1 12:53 .bash_logout
-rw-rwxr-- 1 cactus  ca_group  3771 Jun  1 12:53 .bashrc
-rw-rwxr-- 1 cactus  ca_group   807 Jun  1 12:53 .profile
drwxrwx--- 2 cactus  ca_group  4096 Jun  1 15:02 .ssh
-rw-rwx--- 1 cactus  ca_group 10339 Jun  1 15:55 .viminfo
drwxrwxr-x 2 cactus  ca_group  4096 Jun  1 15:26 csr_requests
drwxrwx--- 3 cactus  ca_group  4096 Jun  1 13:32 easy-rsa
-rw-rwx--- 1 cactus  ca_group  1529 Jun  1 15:55 ukfaresxi02.crt
```

If a user was recenty added to a group, **log them out first for the group to apply**.

<br/>

----

### 1.Install `easy-rsa`

Now we are ready to begin creating our CA server.

First we need to install the `easy-rsa` set of scripts on the CA server. `easy-rsa` being a CA management tool that we will use to generate a **private key**, and **public root certificate**, which we'll then use to sign our CSR requests.

```
lawrence@uk-far-ca-01:~$ sudo apt update
lawrence@uk-far-ca-01:~$ sudo apt install easy-rsa
```

<br/>

### 2.Preparing a PKI directory

Once `easy-rsa` is installed, we need to create a PKI and begin building our CA.

**Do not use sudo** to run any of the following commands. A normal user should be able to manage the CA without elevated privileges. I will create a new user where the CA can be managed and delegate permission to additional users via groups to manage it.

First create a new directory called `easy-rsa` within the CA users `$HOME`. Then create a **symbolic link** to the `easy-rsa` package that we installed in the previous step, this resides in `/usr/share/easy-rsa`:

```
cactus@uk-far-ca-01:~$ mkdir easy-rsa
cactus@uk-far-ca-01:~$ ln -s /usr/share/easy-rsa/* ~/easy-rsa/
```

```
cactus@uk-far-ca-01:~/easy-rsa$ tree
.
├── easyrsa -> /usr/share/easy-rsa/easyrsa
├── openssl-easyrsa.cnf -> /usr/share/easy-rsa/openssl-easyrsa.cnf
├── vars.example -> /usr/share/easy-rsa/vars.example
└── x509-types -> /usr/share/easy-rsa/x509-types
```

---- 

**Heads Up!** Rather than copy the easy-rsa package files into your PKI directory `~/easy-rsa`, use a symlink approach, therefore any updates to the `easy-rsa` **package** will be automatically reflected within our PKI’s scripts.

----

Lastly we'll initialize the PKI inside the `easy-rsa` directory:

```
cactus@uk-far-ca-01:~/easy-rsa$ ./easyrsa init-pki

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /home/cactus/easy-rsa/pki
```

Afterwards we should have a directory containing all the files that are required to build our CA:

```
cactus@uk-far-ca-01:~/easy-rsa$ tree
.
├── easyrsa -> /usr/share/easy-rsa/easyrsa
├── openssl-easyrsa.cnf -> /usr/share/easy-rsa/openssl-easyrsa.cnf
├── pki
│   ├── openssl-easyrsa.cnf
│   ├── private
│   └── reqs
├── vars.example -> /usr/share/easy-rsa/vars.example
└── x509-types -> /usr/share/easy-rsa/x509-types
```

<br/>

### 3.Creating a CA

Before we can create our CA’s **private key** and **root certificate**, we need to create and populate a file called `vars` with some default values.

The `vars` file will contain the settings for the CA information when signing our requests.

```
cactus@uk-far-ca-01:~/easy-rsa$ cp vars.example vars
cactus@uk-far-ca-01:~/easy-rsa$ vim vars
```

I have copied the `vars.example` and amended as follows based on our dept:

```
set_var EASYRSA_REQ_COUNTRY     "UK"
set_var EASYRSA_REQ_PROVINCE    "Greater London"
set_var EASYRSA_REQ_CITY        "London"
set_var EASYRSA_REQ_ORG         "cactus"
set_var EASYRSA_REQ_EMAIL       "techops@cactus.com"
set_var EASYRSA_REQ_OU          "IT Dept"
```

Now to create the **root public** and **private key pair** for our CA by running `easy-rsa` but with the `build-ca` parameter:

```
cactus@uk-far-ca-01:~/easy-rsa$ ./easyrsa build-ca

Note: using Easy-RSA configuration from: ./vars

Using SSL: openssl OpenSSL 1.1.1f  31 Mar 2020

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
Generating RSA private key, 2048 bit long modulus (2 primes)
.............+++++
....................................................+++++
e is 65537 (0x010001)
Can't load /home/cactus/easy-rsa/pki/.rnd into RNG
140377291994432:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:98:Filename=/home/cactus/easy-rsa/pki/.rnd
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:uk-far-ca-01

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/cactus/easy-rsa/pki/ca.crt
```

As you can see I seem to have encountered the following error:

```
Can't load /home/cactus/easy-rsa/pki/.rnd into RNG
140377291994432:error:2406F079:random number generator:RAND_load_file:Cannot open file:../crypto/rand/randfile.c:98:Filename=/home/cactus/easy-rsa/pki/.rnd
```

In this case I can create the file that is missing with the following: 

```
cactus@uk-far-ca-01:~/easy-rsa$ openssl rand -out pki/.rnd -hex 256
```

Now this time it should run successfully:

```
cactus@uk-far-ca-01:~/easy-rsa$ ./easyrsa build-ca

Note: using Easy-RSA configuration from: ./vars

Using SSL: openssl OpenSSL 1.1.1f  31 Mar 2020

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
Generating RSA private key, 2048 bit long modulus (2 primes)
...................+++++
.............+++++
e is 65537 (0x010001)
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:uk-far-ca-01

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/cactus/easy-rsa/pki/ca.crt
```

---- 

**Heads Up!** You'll be prompted to enter a **CA Key Passphrase** for your key pair. Choose a strong passphrase and keep it safe, as you will require it any time that you need to interact with your CA, like for example to sign or revoke a certificate.

----

During the `build-ca` process we will also need to confirm a `CN` or 'Common Name' for our CA. The `CN` is used to refer to this machine in the context of the CA. I have named my `CN` as my servers host name `uk-far-ca-01`.

After that we should have a newly generated `pki/ca.crt` certificate as well as a `pki/private/ca.key` which will make up the **public** and **private** parts of our CA:

```diff
 cactus@uk-far-ca-01:~/easy-rsa$ tree -a
 .
 ├── easyrsa -> /usr/share/easy-rsa/easyrsa
 ├── openssl-easyrsa.cnf -> /usr/share/easy-rsa/openssl-easyrsa.cnf
 ├── pki
 │   ├── .rnd
+│   ├── ca.crt
 │   ├── certs_by_serial
 │   ├── index.txt
 │   ├── issued
 │   ├── openssl-easyrsa.cnf
+│   ├── private
+│   │   └── ca.key
 │   ├── renewed
 │   │   ├── certs_by_serial
 │   │   ├── private_by_serial
 │   │   └── reqs_by_serial
 │   ├── reqs
 │   ├── revoked
 │   │   ├── certs_by_serial
 │   │   ├── private_by_serial
 │   │   └── reqs_by_serial
 │   ├── safessl-easyrsa.cnf
 │   └── serial
 ├── vars
 ├── vars.example -> /usr/share/easy-rsa/vars.example
 └── x509-types -> /usr/share/easy-rsa/x509-types
```

The `ca.crt` is the CA's **public certificate**. This will be used by connecting hosts, to verify that they are part of the same "web of trust", and is therefore required to exist on every user machine so that our SSL handshake can be successful. 

The `ca.key` is the CA's **private key** and will be used to sign our certificate requests. If this file at all becomes compromised, then the CA will need to be **destroyed**. It is therefore recommended that this file should only exist on our CA host, and the host (CA) should be kept **offline** as an extra security measure when not in use.

Now our CA is ready to be used to sign (and revoke) certificates requests.

---

**Heads Up! Remember that the purpose of the CA is to sign and revoke certificates. It is never contacted during the SSL handshake process and therefore does not need to be online for an SSL certificate to be validated.**

---- 

### 4.Distributing our CA's public certificate

For your SSL cert to work, the generated `ca.crt` will need to be added to servers as well as user hosts, so that the identity of another host using our signed SSL certificates, can be verified. 

On a grander scale this would be normally achieved through means of an MDM, whereby the IT department would distribute this CA `.crt` to users machines and then installed within their operating system's certificate store.

In our case, we are going to individually import this `ca.crt` certificate to the relevant hosts that require SSL as a service. This can be done by simply `scp` or copy and pasting the contents of the file to the desired host.

```
cactus@uk-far-ca-01:~/easy-rsa$ scp pki/ca.crt lawrence@10.220.101.117:/tmp/
ca.crt              
```

Now the `ca.crt` can be imported by doing the following (I am using a Debian based system). I rename the `ca.crt` to something that can easily be identified, in this case, `uk-far-ca-01.crt`. 

```
lawrence@cacti:~$ sudo bash
root@cacti:~$ mkdir /usr/local/share/ca-certificates/cactus-ca
root@cacti:~$ mv /tmp/ca.crt /usr/local/share/ca-certificates/cactus-ca/uk-far-ca-01.crt
root@cacti:~$ sudo update-ca-certificates 
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

Now my host machine will trust any certificate that has been signed by our CA server. 

---

**Heads Up!** On an OS such as Ubuntu, browsers such as Firefox and Chrome use their own certificate store. 

**Updating the Ubuntu system certificate store with `update-ca-certificates` has no effect on Chrome or Firefox.**

---- 

<br/>

### 5.Signing a CSR

Now my CA is ready to sign a CSR.

First I will need to generate a certificate request on the remote host:

```
-----BEGIN CERTIFICATE REQUEST-----
MIIDgDCCAmgCAQAwggEAMQswCQYDVQQGEwJVUzETMBEGA1UECAwKQ2FsaWZvcm5p
YTESMBAGA1UEBwwJUGFsbyBBbHRvMRQwEgYDVQQKDAtWTXdhcmUsIEluYzEuMCwG
A1UECwwlVk13YXJlIEVTWCBTZXJ2ZXIgRGVmYXVsdCBDZXJ0aWZpY2F0ZTEqMCgG
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
ne3tUdKIVic9yXtFVtAxGOMEOo0rG+7J9gpNc237PQW6DMjZ
-----END CERTIFICATE REQUEST-----
```

Since I will be operating inside the CA’s PKI where the easy-rsa utility is available, the signing steps will use the easy-rsa utility to make things easier, as opposed to using openssl directly like I did in the previous example.

**You can simply create a text file on the CA and paste the CSR to begin working with it:** 

```
lawrence@uk-far-ca-01:/home/cactus$ vim csr_requests/esxi02.csr
```

The first step is to **import the CSR** with the `easy-rsa` script:

```
lawrence@uk-far-ca-01:/home/cactus$ cd easy-rsa/
lawrence@uk-far-ca-01:/home/cactus/easy-rsa$ ./easyrsa import-req /home/cactus/csr_requests/esxi02.csr uk-far-esxi-02

Note: using Easy-RSA configuration from: ./vars

Using SSL: openssl OpenSSL 1.1.1f  31 Mar 2020

The request has been successfully imported with a short name of: uk-far-esxi-02
You may now use this name to perform signing operations on this request.
```

Now we can **sign the request** by running `easyrsa sign-req` followed by the request type and CN that is included in the CSR. The request type can either be one of `client`, `server`, or `ca`:

```
lawrence@uk-far-ca-01:/home/cactus/easy-rsa$ ./easyrsa sign-req server uk-far-esxi-02

Note: using Easy-RSA configuration from: ./vars

Using SSL: openssl OpenSSL 1.1.1f  31 Mar 2020


You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 1080 days:

subject=
    countryName               = US
    stateOrProvinceName       = California
    localityName              = Palo Alto
    organizationName          = VMware, Inc
    organizationalUnitName    = VMware ESX Server Default Certificate
    emailAddress              = ssl-certificates@vmware.com
    commonName                = uk-far-esxi-02.corp.cactus.com


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from /home/cactus/easy-rsa/pki/safessl-easyrsa.cnf
Enter pass phrase for /home/cactus/easy-rsa/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'US'
stateOrProvinceName   :ASN.1 12:'California'
localityName          :ASN.1 12:'Palo Alto'
organizationName      :ASN.1 12:'VMware, Inc'
organizationalUnitName:ASN.1 12:'VMware ESX Server Default Certificate'
emailAddress          :IA5STRING:'ssl-certificates@vmware.com'
commonName            :ASN.1 12:'uk-far-esxi-02.corp.cactus.com'
Certificate is to be certified until May 17 15:31:34 2023 GMT (1080 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /home/cactus/easy-rsa/pki/issued/uk-far-esxi-02.crt
```

Once that's complete, you'll have a signed CSR which in my case was `pki/reqs/uk-far-esxi-02.req` and was signed by my CA's private key `easy-rsa/pki/private/ca.key`

```
lawrence@uk-far-ca-01:/home/cactus/easy-rsa$ ls pki/reqs/
uk-far-esxi-02.req
```

```
lawrence@uk-far-ca-01:/home/cactus/easy-rsa$ ls pki/private/
ca.key
```

Now I have a **signed certificate** that contains my **VMware server's public encryption key**, as well as a new signature from the CA Server. This signature informs anyone that trusts the CA, can also trust the VMware server's certificate.

```
lawrence@uk-far-ca-01:/home/cactus/easy-rsa$ ls pki/issued/
uk-far-esxi-02.crt
```

Now I can copy my signed certificate to my VMware host:

```
lawrence@uk-far-ca-01:/home/cactus/easy-rsa$ cat pki/issued/uk-far-esxi-02.crt 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            f1:28:47:e9:18:f5:fd:1e:92:ac:a8:a9:82:c0:dc:e2
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN=uk-far-ca-01
        Validity
            Not Before: Jun  1 15:31:34 2020 GMT
            Not After : May 17 15:31:34 2023 GMT
        Subject: C=US, ST=California, L=Palo Alto, O=VMware, Inc, OU=VMware ESX Server Default Certificate, CN=uk-far-esxi-02.corp.cactus.com/emailAddress=ssl-certificates@vmware.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:b7:1d:72:03:e5:b3:20:a1:60:46:0a:50:8e:64:
                    ad:be:e9:75:a2:84:83:fd:48:70:a4:0e:a0:ba:b8:
                    c5:d7:ae:af:4c:aa:b3:64:0a:2f:1e:6f:ea:56:52:
                    7a:c1:f9:ce:57:36:7f:3a:e9:4c:ff:06:74:50:90:
                    .............................................
                    1b:87
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Subject Key Identifier: 
                3C:CC:72:FB:C2:A5:5C:2B:80:28:A7:53:74:5D:2B:24:0B:92:27:7A
            X509v3 Authority Key Identifier: 
                keyid:E3:AD:E2:07:06:4D:56:DB:12:3A:68:E9:4D:56:76:44:1C:D5:26:AD
                DirName:/CN=uk-far-ca-01
                serial:69:64:EE:FF:40:39:F3:4C:37:41:36:76:55:4A:EF:8E:18:84:C3:61

            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Key Usage: 
                Digital Signature, Key Encipherment
            X509v3 Subject Alternative Name: 
                DNS:uk-far-esxi-02.corp.cactus.com
    Signature Algorithm: sha256WithRSAEncryption
         b1:8e:90:87:bc:c5:c5:29:03:83:f7:80:64:07:35:f5:cd:b1:
         0f:70:31:76:47:90:d7:0a:c5:25:fd:19:a7:b9:7f:74:ec:b9:
         2c:30:13:7b:49:db:a9:da:7e:1b:bd:86:46:fe:cd:86:3e:9a:
         .....................................................
         5f:2d:19:f7
-----BEGIN CERTIFICATE-----
MIIEPTCCAyWgAwIBAgIRAPEoR+kY9f0ekqyoqYLA3OIwDQYJKoZIhvcNAQELBQAw
FzEVMBMGA1UEAwwMdWstZmFyLWNhLTAxMB4XDTIwMDYwMTE1MzEzNFoXDTIzMDUx
NzE1MzEzNFowgc4xCzAJBgNVBAYTAlVTMRMwEQYDVQQIDApDYWxpZm9ybmlhMRIw
EAYDVQQHDAlQYWxvIEFsdG8xFDASBgNVBAoMC1ZNd2FyZSwgSW5jMS4wLAYDVQQL
................................................................
-----END CERTIFICATE-----
```

All issued certs are stored within `/home/cactus/easy-rsa/pki/issued/`:

```
lawrence@uk-far-ca-01:/home/cactus/easy-rsa$ls pki/issued/
uk-far-esxi-02.crt
```

To be able to use it with my ESXi server, I will need to convert it to `.pem` format (a Linux format that my VMware host requires). In which case I can use the generated `.pem` from `pki/certs_by_serial`:

```
├── pki
│   ├── ca.crt
│   ├── certs_by_serial
│   │   └── F12847E918F5FD1E92ACA8A982C0DCE2.pem
```

And that's it. It's that simple.

<br/>

----- 

### Testing your SSL certs

You can test an SSL connection with the `openssl s_client`:

```
lawrence@cacti:~$ openssl s_client -connect uk-far-esxi-02.corp.cactus.com:443 -CApath /etc/ssl/certs/
CONNECTED(00000007)
depth=0 C = US, ST = California, L = Palo Alto, O = "VMware, Inc", OU = VMware ESX Server Default Certificate, CN = uk-far-esxi-02.corp.cactus.com, emailAddress = ssl-certificates@vmware.com
verify error:num=20:unable to get local issuer certificate
verify return:1
depth=0 C = US, ST = California, L = Palo Alto, O = "VMware, Inc", OU = VMware ESX Server Default Certificate, CN = uk-far-esxi-02.corp.cactus.com, emailAddress = ssl-certificates@vmware.com
verify error:num=21:unable to verify the first certificate
verify return:1
---
Certificate chain
 0 s:C = US, ST = California, L = Palo Alto, O = "VMware, Inc", OU = VMware ESX Server Default Certificate, CN = uk-far-esxi-02.corp.cactus.com, emailAddress = ssl-certificates@vmware.com
   i:CN = uk-far-ca-01
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEPTCCAyWgAwIBAgIRAPEoR+kY9f0ekqyoqYLA3OIwDQYJKoZIhvcNAQELBQAw
FzEVMBMGA1UEAwwMdWstZmFyLWNhLTAxMB4XDTIwMDYwMTE1MzEzNFoXDTIzMDUx
NzE1MzEzNFowgc4xCzAJBgNVBAYTAlVTMRMwEQYDVQQIDApDYWxpZm9ybmlhMRIw
EAYDVQQHDAlQYWxvIEFsdG8xFDASBgNVBAoMC1ZNd2FyZSwgSW5jMS4wLAYDVQQL
DCVWTXdhcmUgRVNYIFNlcnZlciBEZWZhdWx0IENlcnRpZmljYXRlMSQwIgYDVQQD
DBt1ay1mYXItZXN4aS0wMi5jb3JwLm1vby5jb20xKjAoBgkqhkiG9w0BCQEWG3Nz
................................................................
-----END CERTIFICATE-----
subject=C = US, ST = California, L = Palo Alto, O = "VMware, Inc", OU = VMware ESX Server Default Certificate, CN = uk-far-esxi-02.corp.cactus.com, emailAddress = ssl-certificates@vmware.com

issuer=CN = uk-far-ca-01

---
No client certificate CA names sent
Peer signing digest: SHA512
Peer signature type: RSA
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 1564 bytes and written 455 bytes
Verification error: unable to verify the first certificate
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
    Master-Key: B333307850B1F580770B4B76655E29637
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1591028744
    Timeout   : 7200 (sec)
    Verify return code: 21 (unable to verify the first certificate)
    Extended master secret: no
---
read:errno=0
```

As you can see by the following error, it **failed**:

```
    Verify return code: 21 (unable to verify the first certificate)
```

This was due the permissions being wrong for the original CA cert:

```
lawrence@cacti:~$ ls -al /etc/ssl/certs/ | grep uk-far-ca-01.crt
lrwxrwxrwx 1 root root     50 Jun  3 19:02 uk-far-ca-01.pem -> /usr/local/share/ca-certificates/cactus-ca/uk-far-ca-01.crt
```

And if I check the original file you'll notice that only root can read write `-rw-------`:

```
lawrence@cacti:~$ ls -la /usr/local/share/ca-certificates/cactus-ca/
-rw------- 1 root root 1208 Jun  1 16:04 ca.crt
```

So I'll fix this by giving `group` and `others` read access:

```
lawrence@cacti:~$ sudo chmod 644 /usr/local/share/ca-certificates/cactus-ca/uk-far-ca-01.crt
lawrence@cacti:~$ ls -la /usr/local/share/ca-certificates/cactus-ca
-rw-r--r-- 1 root root 1208 Jun  1 16:04 uk-far-ca-01.crt
```

And now that I've fixed the permissions with my certificate, I receive `return code: 0 (ok)` implying it worked:

```
lawrence@cacti:~$ openssl s_client -connect uk-far-esxi-02.corp.cactus.com:443 -CApath /etc/ssl/certs/
CONNECTED(00000007)
depth=1 CN = uk-far-ca-01
verify return:1
depth=0 C = US, ST = California, L = Palo Alto, O = "VMware, Inc", OU = VMware ESX Server Default Certificate, CN = uk-far-esxi-02.corp.cactus.com, emailAddress = ssl-certificates@vmware.com
verify return:1
---
Certificate chain
 0 s:C = US, ST = California, L = Palo Alto, O = "VMware, Inc", OU = VMware ESX Server Default Certificate, CN = uk-far-esxi-02.corp.cactus.com, emailAddress = ssl-certificates@vmware.com
   i:CN = uk-far-ca-01
---
Server certificate
Server certificate
-----BEGIN CERTIFICATE-----
MIIEPTCCAyWgAwIBAgIRAPEoR+kY9f0ekqyoqYLA3OIwDQYJKoZIhvcNAQELBQAw
FzEVMBMGA1UEAwwMdWstZmFyLWNhLTAxMB4XDTIwMDYwMTE1MzEzNFoXDTIzMDUx
NzE1MzEzNFowgc4xCzAJBgNVBAYTAlVTMRMwEQYDVQQIDApDYWxpZm9ybmlhMRIw
EAYDVQQHDAlQYWxvIEFsdG8xFDASBgNVBAoMC1ZNd2FyZSwgSW5jMS4wLAYDVQQL
DCVWTXdhcmUgRVNYIFNlcnZlciBEZWZhdWx0IENlcnRpZmljYXRlMSQwIgYDVQQD
DBt1ay1mYXItZXN4aS0wMi5jb3JwLm1vby5jb20xKjAoBgkqhkiG9w0BCQEWG3Nz
................................................................
-----END CERTIFICATE-----
subject=C = US, ST = California, L = Palo Alto, O = "VMware, Inc", OU = VMware ESX Server Default Certificate, CN = uk-far-esxi-02.corp.cactus.com, emailAddress = ssl-certificates@vmware.com

issuer=CN = uk-far-ca-01

---
No client certificate CA names sent
Peer signing digest: SHA512
Peer signature type: RSA
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 1564 bytes and written 455 bytes
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
    Master-Key: B333307850B1F580770B4B76655E29637
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    Start Time: 1591031702
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: no
---
read:errno=0
```

<br/>

### Adding the CA cert to Google Chrome

Chrome by default uses its own certificate store on Ubuntu, therefore to add it, do the following:

Install `libnss3-tools`:

```
lawrence@cacti:~$ sudo apt-get install libnss3-tools
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  ieee-data libllvm7 python-certifi python-jmespath python-kerberos python-libcloud python-lockfile python-netaddr python-requests python-selinux python-urllib3 python-xmltodict
Use 'sudo apt autoremove' to remove them.
The following NEW packages will be installed
  libnss3-tools
0 to upgrade, 1 to newly install, 0 to remove and 11 not to upgrade.
Need to get 870 kB of archives.
After this operation, 4,250 kB of additional disk space will be used.
Get:1 http://gb.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 libnss3-tools amd64 2:3.35-2ubuntu2.7 [870 kB]
Fetched 870 kB in 0s (4,411 kB/s)     
Selecting previously unselected package libnss3-tools.
(Reading database ... 246684 files and directories currently installed.)
Preparing to unpack .../libnss3-tools_2%3a3.35-2ubuntu2.7_amd64.deb ...
Unpacking libnss3-tools (2:3.35-2ubuntu2.7) ...
Setting up libnss3-tools (2:3.35-2ubuntu2.7) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

Add the CA certificate using `certutil`:

```
lawrence@cacti:~$ certutil -d sql:$HOME/.pki/nssdb -A -t "CP,CP," -n uk-far-ca-01 -i uk-far-ca-01.crt
```

You can list out the installed certificate to confirm:

```
lawrence@cacti:~$ certutil -d sql:/home/lawrence/.pki/nssdb -L

Certificate Nickname                                         Trust Attributes
                                                             SSL,S/MIME,JAR/XPI

uk-far-ca-01                                                 CP,C,
```

Lastly, quit Chrome:

```
lawrence@cacti:~$ sudo pkill chrome 
```

The key was using `"CP,CP,"` - everything was failing with Chrome up until that point:

```
certutil -d sql:$HOME/.pki/nssdb -A -t "CP,CP," -n uk-far-ca-01 -i uk-far-ca-01.crt
```

This addresses the NSS bug as kindly mentioned by some blessed soul here: 

https://superuser.com/questions/104146/add-permanent-ssl-certificate-exception-in-chrome-linux/586064#586064

