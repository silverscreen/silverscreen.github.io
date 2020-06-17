---
layout: post
title: Setting up a webhook listener in AWS
description: Setting up a webhook listener in AWS
summary: Setting up a webhook listener in AWS
comments: false
---

I'll be using [adnanh's webhook](https://github.com/adnanh/webhook) tool to configure an agent that will run on my VPS (in this case an EC2 instance running Ubuntu 18.04 in AWS). 

**Disclaimer** ALL **PUBLIC IP addresses** have been modified and are therefore not representational of real life. 

----

### Sources

[Julio Gomez](https://blogs.cisco.com/author/juliogomez)(a Programmability Lead at CISCO) has produced a great set of blog posts around infrastructure programmability and is well worth reading.

----

### Installation

If you are using Ubuntu linux (17.04 or later), you can install webhook using `sudo apt-get install webhook` which will install the community packaged version. Alternatively, it can be installed via `snap` https://snapcraft.io/webhook. 

```
ubuntu@ubuntubox:~$ sudo apt-get install webhook
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  webhook
0 upgraded, 1 newly installed, 0 to remove and 52 not upgraded.
Need to get 1443 kB of archives.
After this operation, 4979 kB of additional disk space will be used.
Get:1 http://eu-west-2.ec2.archive.ubuntu.com/ubuntu bionic/universe amd64 webhook amd64 2.5.0-2 [1443 kB]
Fetched 1443 kB in 0s (6703 kB/s)
Selecting previously unselected package webhook.
(Reading database ... 67532 files and directories currently installed.)
Preparing to unpack .../webhook_2.5.0-2_amd64.deb ...
Unpacking webhook (2.5.0-2) ...
Setting up webhook (2.5.0-2) ...
Created symlink /etc/systemd/system/multi-user.target.wants/webhook.service → /lib/systemd/system/webhook.service.
```

```
ubuntu@ubuntubox:~/webhook$ whereis webhook
webhook: /usr/bin/webhook
```

Then a directory to store my webhooks:

```
ubuntu@ubuntubox:~$ mkdir webhook
```

<br/>

### Creating your first webhook

For hook definitions https://github.com/adnanh/webhook/wiki/Hook-Definition. 

**Hooks are defined as JSON objects, and **must contain** the `id` and `execute-command` properties.**

First identify the webhook with an `id`, this will form part of the HTTP endpoind when making our POST requests i.e. http://22.170.222.179:9001/hooks/cleanup-webhook. 

Secondly I'll specify an `execute-command` which in this case is a script in `/var/scripts/` to be executed when the hook is triggered.

```
ubuntu@ubuntubox:~$ cat webhook/hooks.json                                                              
[   
  {     
    "id": "cleanup-webhook",     
    "execute-command": "/var/scripts/cleanup.sh",     
    "command-working-directory": "/tmp"   
  } 
]
```

Lastly the `command-working-directory` specifies the working directory that will be used for the script when it's executed.


<br/>

#### Confirming your webhook is running


Looks good. So now I'll proceed with spawning my webhook service: 

```
ubuntu@ubuntubox:~/webhook$ webhook -hooks hooks.json -verbose -ip "22.170.222.179"
[webhook] 2019/11/21 17:06:25 version 2.5.0 starting
[webhook] 2019/11/21 17:06:25 setting up os signal watcher
[webhook] 2019/11/21 17:06:25 attempting to load hooks from hooks.json
[webhook] 2019/11/21 17:06:25 found 1 hook(s) in file
[webhook] 2019/11/21 17:06:25   loaded: cleanup-webhook
[webhook] 2019/11/21 17:06:25 serving hooks on http://22.170.222.179:9000/hooks/{id}
[webhook] 2019/11/21 17:06:25 listen tcp 22.170.222.179:9000: bind: cannot assign requested address
```

Here I'm receiving the error `listen tcp 22.170.222.179:9000: bind: cannot assign requested address`. 

If I check what's running on port `9000`:

```
ubuntu@ubuntubox:~/webhook$ sudo netstat -pna | grep 9000
tcp6       0      0 :::9000                 :::*                    LISTEN      9763/webhook
```

By the looks I've of it, I have a IPv6 service running hence the `tcp6`.

So I'll kill that service:

```
ubuntu@ubuntubox:~/webhook$ sudo kill -9 9763
```

And then confirm nothing is running on that port:

```
ubuntu@ubuntubox:~/webhook$ sudo netstat -pna | grep 9000

```

Now I can proceed to running my webhook:

```
ubuntu@ubuntubox:~/webhook$ webhook -hooks hooks.json -verbose
[webhook] 2019/11/21 17:06:49 version 2.5.0 starting
[webhook] 2019/11/21 17:06:49 setting up os signal watcher
[webhook] 2019/11/21 17:06:49 attempting to load hooks from hooks.json
[webhook] 2019/11/21 17:06:49 found 1 hook(s) in file
[webhook] 2019/11/21 17:06:49   loaded: cleanup-webhook
[webhook] 2019/11/21 17:06:49 serving hooks on http://0.0.0.0:9000/hooks/{id}
[webhook] 2019/11/21 17:06:49 os signal watcher ready

[webhook] 2019/11/21 17:07:11 Started POST /hooks/cleanup-webhook
[webhook] 2019/11/21 17:07:11 cleanup-webhook got matched
[webhook] 2019/11/21 17:07:11 cleanup-webhook hook triggered successfully
[webhook] 2019/11/21 17:07:11 Completed 200 OK in 669.473µs
[webhook] 2019/11/21 17:07:11 executing /var/scripts/cleanup.sh (/var/scripts/cleanup.sh) with arguments ["/var/scripts/cleanup.sh"] and environment [] using /tmp as cwd
[webhook] 2019/11/21 17:07:11 command output:
[webhook] 2019/11/21 17:07:11 finished handling cleanup-webhook
```

I can run on a different port but using the `-port` switch: 

```
~/webhook$ webhook -hooks hooks.json -verbose -port 9001
```

<br/>

----

#### Running a process in the background

Currently I have no jobs running: 

```
ubuntu@ubuntubox:~$ jobs
```

And right now the service is also down:

```                              
ubuntu@ubuntubox:~$ sudo service webhook status                                                                                          
● webhook.service - Small server for creating HTTP endpoints (hooks)
   Loaded: loaded (/lib/systemd/system/webhook.service; enabled; vendor preset: enabled)
   Active: failed (Result: signal) since Mon 2019-11-25 12:13:24 UTC; 4 months 22 days ago
     Docs: https://github.com/adnanh/webhook/
 Main PID: 883 (code=killed, signal=KILL)

Nov 25 12:07:00 ubuntubox systemd[1]: Started Small server for creating HTTP endpoints (hooks).
Nov 25 12:10:51 ubuntubox webhook[883]: [webhook] 2019/11/25 12:10:51 Started POST /hooks/cleanup-webhook
Nov 25 12:10:51 ubuntubox webhook[883]: [webhook] 2019/11/25 12:10:51 Completed 404 Not Found in 144.098µs
Nov 25 12:12:07 ubuntubox webhook[883]: [webhook] 2019/11/25 12:12:07 Started POST /hooks/cleanup-webhook
Nov 25 12:12:07 ubuntubox webhook[883]: [webhook] 2019/11/25 12:12:07 Completed 404 Not Found in 40.373µs
Nov 25 12:12:58 ubuntubox webhook[883]: [webhook] 2019/11/25 12:12:58 Started POST /hooks/cleanup-webhook
Nov 25 12:12:58 ubuntubox webhook[883]: [webhook] 2019/11/25 12:12:58 Completed 404 Not Found in 130.862µs
Nov 25 12:13:24 ubuntubox systemd[1]: webhook.service: Main process exited, code=killed, status=9/KILL
Nov 25 12:13:24 ubuntubox systemd[1]: webhook.service: Failed with result 'signal'.
```

So I'll start my service:

```
ubuntu@ubuntubox:~$ sudo service webhook start
ubuntu@ubuntubox:~$                                                                                       
ubuntu@ubuntubox:~$ sudo service webhook status
● webhook.service - Small server for creating HTTP endpoints (hooks)
   Loaded: loaded (/lib/systemd/system/webhook.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-04-17 10:04:41 UTC; 2s ago
     Docs: https://github.com/adnanh/webhook/
 Main PID: 30747 (webhook)
    Tasks: 4 (limit: 1152)
   CGroup: /system.slice/webhook.service
           └─30747 /usr/bin/webhook -nopanic -hooks /etc/webhook.conf

Apr 17 10:04:41 ubuntubox systemd[1]: Started Small server for creating HTTP endpoints (hooks).
````

And now I'll detach a process from my terminal and run it in the background by appending it with an ampersand `&` sign:

```
ubuntu@ubuntubox:~$ webhook -hooks webhook/hooks.json -verbose -port 9001 &         
[1] 30812
ubuntu@ubuntubox:~$ [webhook] 2020/04/17 10:13:24 version 2.5.0 starting
[webhook] 2020/04/17 10:13:24 setting up os signal watcher
[webhook] 2020/04/17 10:13:24 attempting to load hooks from webhook/hooks.json
[webhook] 2020/04/17 10:13:24 found 1 hook(s) in file
[webhook] 2020/04/17 10:13:24   loaded: cleanup-webhook
[webhook] 2020/04/17 10:13:24 serving hooks on http://0.0.0.0:9001/hooks/{id}
[webhook] 2020/04/17 10:13:24 os signal watcher ready
```

Running a process in your terminal can sometimes result in the following problems: 
* The controlling terminal is filled with output data and error/diagnostic messages.
* In the event that the terminal is closed, the process together with its child processes will be terminated.

For these reasons, it is desirable to detach the process.

https://www.tecmint.com/run-linux-command-process-in-background-detach-process/

You can view all your background jobs by typing `jobs`:

```
ubuntu@ubuntubox:~$ jobs
[1]+  Running                 webhook -hooks webhook/hooks.json -verbose -port 9001 &
```

Now I'll attempt to setup and trigger my webhook from GitHub: 

<br/>

----

### Configure a webhook payload in GitHub

Before I proceed with testing my webhook, I'll need to ensure that the relevant ports are open for my EC2 instance otherwise my webhook listener wont be able to receive the payload from GitHub. 

I have opened the following ports in AWS under **EC2>Security Groups** and added an inbound rule to my security group to allow **TCP, 9000-9001, 0.0.0.0/0**.

Webhooks allow external services to be notified when certain events happen. When the specified events happen, GitHub will send a POST request to each of the URLs provided.

Add the webhook within the relevant GitHub repo under **Settings>Webhooks>Add webhook**.

I have configured the following:

| Payload URL                                      | Content type     | Secret |  Events       |
|--------------------------------------------------|------------------|--------|---------------------------------|
| http://22.170.222.179:9001/hooks/cleanup-webhook | application/json |        |  `push` event |

Once configured, I can then make my webhook active, at which point it clicking **Update webhook** will make a POST request to confirm the webhook is reachable. 

If everything is successful, underneath 'Recent Deliveries' you should receive a HTTP 200 OK success status response code indicating that everything is working.

Now on the linux box that I configured the webhook listener on, I can see in the console that a match occurred and that the webhook `triggered successfully`:

```
ubuntu@ubuntubox:~$ 
ubuntu@ubuntubox:~$ [webhook] 2020/04/17 10:18:51 Started POST /hooks/cleanup-webhook
[webhook] 2020/04/17 10:18:51 cleanup-webhook got matched
[webhook] 2020/04/17 10:18:51 cleanup-webhook hook triggered successfully
[webhook] 2020/04/17 10:18:51 Completed 200 OK in 2.824382ms
[webhook] 2020/04/17 10:18:51 executing /var/scripts/cleanup.sh (/var/scripts/cleanup.sh) with arguments ["/var/scripts/cleanup.sh"] and environment [] using /tmp as cwd
[webhook] 2020/04/17 10:18:51 command output: 
[webhook] 2020/04/17 10:18:51 error occurred: exit status 1
[webhook] 2020/04/17 10:18:51 finished handling cleanup-webhook
```

Cheers!

----