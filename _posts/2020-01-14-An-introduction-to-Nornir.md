---
layout: post
title: An introduction to Nornir
description: An introduction to Nornir
summary: An introduction to Nornir
comments: false
---

Nornir is an automation framework written in python and can be imported like any other python library. 

#### At A Glance 

Nornir is an automation framework written in python, for python, rather than having its own configuration language. As nornir allows you to use pure python code, you can troubleshoot and debug it in the same way as you would do with any other python code. Only requires a basic understanding of python to use, therefore you should have an understanding of basic concepts such as variables, functions, and imports.

----- 

### Installation

Nornir is available through PyPI and can be installed like most other python packages using pip.

First install nornir using `pip3 install nornir`:

```
lawrence@ulysses:~/git/Junos/automation/ansible$ sudo pip3 install nornir
[sudo] password for lawrence: 
Collecting nornir
  Downloading https://files.pythonhosted.org/packages/b2/7d/d5765da3ef32ec44f7d4e41321542c9956df18eedd7864f0e8f1409a73e3/nornir-2.4.0-py3-none-any.whl (136kB)
    100% |████████████████████████████████| 143kB 1.9MB/s 
[...]
Installing collected packages: mypy-extensions, ruamel.yaml.clib, ruamel.yaml, typing-extensions, dataclasses, paramiko, future, textfsm, netmiko, pyeapi, pyIOSXR, dnspython, passlib, ciscoconfparse, nxapi-plumbing, napalm, nornir
  Found existing installation: paramiko 2.5.0
    Uninstalling paramiko-2.5.0:
      Successfully uninstalled paramiko-2.5.0
  Running setup.py install for future ... done
  Running setup.py install for pyeapi ... done
  Running setup.py install for pyIOSXR ... done
  Running setup.py install for nxapi-plumbing ... done
Successfully installed ciscoconfparse-1.4.11 dataclasses-0.7 dnspython-1.16.0 future-0.18.2 mypy-extensions-0.4.3 napalm-2.5.0 netmiko-3.0.0 nornir-2.4.0 nxapi-plumbing-0.5.2 paramiko-2.7.1 passlib-1.7.2 pyIOSXR-0.53 pyeapi-0.8.3 ruamel.yaml-0.16.10 ruamel.yaml.clib-0.2.0 textfsm-1.1.0 typing-extensions-3.7.4.1
```

You'll notice that it installs a few packages including netmiko and napalm. This, when coupled with the appropriate vendor libraries to manage your devices, results in a complete package to manage your network through python.  

Now that nornir is installed, we can go ahead and import the package from python.

```
python
>>>import nornir.core
>>>
```

<br/>

### Configuration

Nornir is comprised of a config file, and several inventory files which we refer to within our main config file:

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/ipvzero$ tree
.
├── config.yaml
├── inventory
│   ├── defaults.yaml
│   ├── groups.yaml
│   └── hosts.yaml
├── nornir.log
```

<br/>

### Configuration | Config

With `InitNornir` you can initialize nornir with a configuration file, code, or both. 

Within the `config.yaml` file we can pass a dictionary of options for each section, setting the relevant environment variables or by a combination of the three, with the order of preference from less to more preferred "config file" -> "env variable" -> "code".

Our yaml config file should look something like this:

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/ipvzero$ cat config.yaml 
---
core: 
    num_workers: 50

inventory:
    plugin: nornir.plugins.inventory.simple.SimpleInventory
    options:
        host_file: "/home/lawrence/git/Junos/automation/python/nornir/ipvzero/hosts.yaml"
        group_file: "/home/lawrence/git/Junos/automation/python/nornir/ipvzero/groups.yaml"
        defaults_file: "/home/lawrence/git/Junos/automation/python/nornir/ipvzero/defaults.yaml"
```

If we were to write this in python without a configuration file:

```
from nornir import InitNornir
nr = InitNornir(
    core={"num_workers": 50},
    inventory={
        "plugin": "nornir.plugins.inventory.simple.SimpleInventory",
        "options": {
            "host_file": "/home/lawrence/git/Junos/automation/python/nornir/ipvzero/hosts.yaml",
            "group_file": "inventory/groups.yaml"
        }
    }
)
```

So now that we have a basic configuration file, we can begin populating our inventory with devices.

<br/> 

### Configuration | Inventory

An inventory typically consists of hosts, groups and defaults. I'll be using the SimpleInventory plugin, which stores the relevant data in the three aforementioned files.

**The GROUPS and DEFAULTS files are infact optional, but the HOSTS file is always required.**

The schema of SimpleInventory plugin:

```
classnornir.plugins.inventory.simple.SimpleInventory(host_file: str = 'hosts.yaml', group_file: str = 'groups.yaml', defaults_file: str = 'defaults.yaml', hosts: Optional[Dict[str, Dict[str, Any]]] = None, groups: Optional[Dict[str, Dict[str, Any]]] = None, defaults: Optional[Dict[str, Any]] = None, *args, **kwargs)
```

We begin describing our outermost key which is the name of the host, and then an `InventoryElemement` object. The schema of the object can be seen below by executing: 

```
>>> from nornir.core.deserializer.inventory import InventoryElement
>>> import json
>>> print(json.dumps(InventoryElement.schema(), indent=4))

{
    "title": "InventoryElement",
    "type": "object",
    "properties": {
        "hostname": {
            "title": "Hostname",
            "type": "string"
        },
        "port": {
            "title": "Port",
            "type": "integer"
        },
        "username": {
            "title": "Username",
            "type": "string"
        },
        "password": {
            "title": "Password",
            "type": "string"
        },
        "platform": {
            "title": "Platform",
            "type": "string"
        },
        "groups": {
            "title": "Groups",
            "default": [],
            "type": "array",
            "items": {
                "type": "string"
            }
        },
        "data": {
            "title": "Data",
            "default": {},
            "type": "object"
        },
        "connection_options": {
            "title": "Connection Options",
            "default": {},
            "type": "object",
            "additionalProperties": {
                "$ref": "#/definitions/ConnectionOptions"
            }
        }
    },
    "definitions": {
        "ConnectionOptions": {
            "title": "ConnectionOptions",
            "type": "object",
            "properties": {
                "hostname": {
                    "title": "Hostname",
                    "type": "string"
                },
                "port": {
                    "title": "Port",
                    "type": "integer"
                },
                "username": {
                    "title": "Username",
                    "type": "string"
                },
                "password": {
                    "title": "Password",
                    "type": "string"
                },
                "platform": {
                    "title": "Platform",
                    "type": "string"
                },
                "extras": {
                    "title": "Extras",
                    "type": "object"
                }
            }
        }
    }
}
```

Within my `hosts.yaml` example, I am using keybased ssh and therefore no password is needed, instead I specify that netmiko `use_keys: True` which will use my public ssh key to authenticate the connection:

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/ipvzero$ cat inventory/hosts.yaml
---
R1: 
    hostname: 192.168.10.21
    platform: junos
    username: lawrence
    connection_options:
      netmiko:
        extras:
          use_keys: True
    groups: 
        - junos_group
    data: 
        site: london
        role: spine
        type: network_device
```            

The `groups_file` follows the same rules as the `hosts_file`: 

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/ipvzero$ cat inventory/groups.yaml 
---

junos_group:
    platform: junos
```    

And finally the defaults file, which follows the same schema as the `InventoryElement` before, but without outer keys to denote individual elements. 

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/ipvzero$ cat inventory/defaults.yaml 

[need to build this]

```

<br/>

### Accessing the inventory

The inventory can be accessed with the `inventory` attribute. 

The inventory has two dictionary-like attributes `hosts` and `groups` that can be used to access the hosts and groups respectively:

```
>>> from nornir import InitNornir
>>> nr = InitNornir(config_file="config.yaml")
>>>
>>> print(nr.inventory.hosts)
{'R1': Host: R1}
>>>
>>> print(nr.inventory.groups)
{'global': Group: global, 'junos_group': Group: junos_group}
```

```
>>> print(nr.inventory.groups["junos_group"])
junos_group
>>> print(nr.inventory.hosts["R1"])
R1
```

Hosts and groups are also dict-like objects:

```
>>> host = nr.inventory.hosts["R1"]
>>> host.keys()
dict_keys(['site', 'role', 'type'])
>>>
>>> host["site"]
'london'
>>>
>>> host["type"]
'network_device'
```

<br/>

### Inheritance model

* https://nornir.readthedocs.io/en/stable/tutorials/intro/inventory.html#Inheritance-model [12]

Looking at the groups `inventory/groups.yaml` file:

```
---
global:
    data:
        domain: global.local

farringdon:
    data:
        ntp: 
            servers:
                - 4.5.6.7 
    groups:
        - junos_group

aws:
    data:
        domain: aws.global

aws_platform:
    data:
        asn: 9059
    groups:
        - aws 

aws_it:
    data:
        asn: 64512
    groups:
        - aws
        
junos_group:
    username: lawrence
    platform: junos
    data:
        asn: 65010
    groups:  
        - global

```

And my device described within my `inventory/hosts.yaml` file: 

```
---
R1: 
    hostname: 192.168.10.21
    platform: junos
    username: lawrence
    connection_options:
        netmiko:
            extras:
                use_keys: True
    groups: 
        - farringdon
    data: 
        site: london
        role: spine
        type: network_device
```


My host device `R1` exists within the group `farringdon`, which in turn belongs to the group `junos_group` and finally as part of the global group `global`. 

One of the advantages of nornir vs ansible is that a device/group can exist within multiple groups. 

```
>>> from nornir import InitNornir
>>> nr = InitNornir(config_file="config.yaml")
>>> print(nr.inventory.groups)
{'global': Group: global, 'farringdon': Group: farringdon, 'aws': Group: aws, 'aws_platform': Group: aws_platform, 'aws_it': Group: aws_it, 'junos_group': Group: junos_group}
```  

In this example you can see how data resolution works by iterating recursively over all the parent groups and trying to see if that parent group (or any of its parents) contain the data that's being queried. 

For instance, device `R1` belongs to `farringdon`, which is part of `junos_group`, and finally `global`. In our example we are querying `"domain"`, information of which is stored in the top group `global`.   

```
>>> from nornir import InitNornir
>>> nr = InitNornir(config_file="config.yaml")
>>> gns3r1 = nr.inventory.hosts["R1"]
>>> gns3r1["domain"]
'global.local'
```

If however I added the field `domain: nornir.test` to the group `junos_group`, then it would have stopped interating through all parent groups (`global`) and instead returned the `domain: nornir.test` data from `junos_group`.

<br/>

---- 

### `defaults.yaml` and custom inventory data

Values in `defaults` will be returned if neither the host nor the parents have a specific value for it, for example:

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/ipvzero$ cat inventory/defaults.yaml 
---
data:
    cat_breed: sphynx
```    

Here I added a **new data key** in `inventory/defaults.yaml` which I named `cat_breed:`. When I query this new data key, nornir iterates through each group attempting to return the value for this key, upon last resort it finally checks the `defaults.yaml` file and returns the 'default' value for that key `sphynx`: 

```
>>> gns3r1["cat_breed"]
'sphynx'
```

Any custom host or group data keys, other than core supported keys, must exist under a data subkey:

```
--- 
global:
    data:
        domain: gns3lab.local

junos_group:
    username: lawrence
    platform: junos
    data:
        asn: 65010
    connection_options:
        netmiko:
            extras:
                use_keys: True
    groups:
        - global

farringdon_firewalls:
    data:
        country: uk
        site: farringdon
        type: firewall
        ntp:
            servers: 
                - 80.86.38.193 
    groups: 
        - junos_group
```

<br/>

---- 

###  Accessing Custom `data` Attributes

Here's a host from my `hosts.yaml` that has a custom data key `vrrp-group-1`:

```
MOO-UK-FAR-SRX2:
    hostname: 10.130.2.253
    groups:
        - farringdon_firewalls
    platform: junos
    data:
        site: farringdon
        type: srx 
        vrrp-group-1: 
            vip: 149.11.141.138
            address: 149.11.141.141/29
            priority: 90 
```            

And here is the group it belongs to within `groups.yaml`:

```
farringdon_firewalls:
    data:
        country: uk
        site: farringdon
        type: firewall
        ntp:
            servers:
                - 80.86.38.193
        vrrp-group-1: 
            vip: 
                - 149.11.141.138
```                

As you can see both files contain the same `data` key `vrrp-group-1`. I'll demonstrate why this creates a problem.

```
>>> srx2 = nr.inventory.hosts["MOO-UK-FAR-SRX2"]
>>> srx2.keys()
dict_keys(['site', 'type', 'vrrp-group-1', 'country', 'ntp', 'asn', 'domain'])
```

As mentioned previously, nornir iterates through the inventory files in the following order: first `hosts.yaml`, then `groups.yaml` and then finally `defaults.yaml`. 

In this instance it is looking for the data subkey `vrrp-group-1`, therefore the first place it checks is within `hosts.yaml`, it finds a match, and doesn't interate through anymore files for this information:

```
>>> srx2['vrrp-group-1']                            
{'address': '149.11.141.141/29', 'priority': 90}
```

This is why the following sub-elements from our dict-like object return only `address` and `priority`, but not `vip` which is stored within the same data subkey attribute but inside `groups.yaml`. The custom subkey is matched within the host file and immediately stops looking and so the value for `vip` is never returned.

Even calling the sub element within the subkey will fail: 

```
>>> srx2['vrrp-group-1']['vip']                                                                                                               
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'vip'
```

Therefore based on this behaviour, it is strongly advised against populating hosts and groups with conflicting data subkeys.

To mitigate this problem, I've moved the conflicting subkey from `groups.yaml` to `hosts.yaml`. The only caveat is that it's an extra line to repeat in my host file, however it also makes more sense to group attributes around a common subkey. 

```
MOO-UK-FAR-SRX2:
    hostname: 10.130.2.253
    groups:
        - farringdon_firewalls
    platform: junos
    data:
        site: farringdon
        type: srx 
        vrrp-group-1: 
            vip: 149.11.141.138
            address: 149.11.141.141/29
            priority: 90 
```

I'm able to then call the sub-elements from the dict-like objects with:

```
>>> srx2['vrrp-group-1']['address']
'149.11.141.141/29'
>>> srx2['vrrp-group-1']['priority']                               
90
```

More information on accessing sub-elements from a dictionary here: https://stackoverflow.com/questions/25626089/python-dictionary-sub-element-access

<br/>

---- 

### Running your first script

* https://nornir.readthedocs.io/en/latest/tutorials/intro/executing_tasks.html

Here I've created a task that simply print out the hostname site of each device: 

```
from nornir import InitNornir
from nornir.plugins.tasks import networking, text
from nornir.plugins.functions.text import print_title, print_result
from nornir.core.filter import F

nr = InitNornir(config_file="config.yaml", dry_run=True)
dev = nr.filter(F(platform="junos"))


def hi(task):
    print(f"hi! My name is {task.host.name} and I live in {task.host['site']}")

nr.run(task=hi, num_workers=1)
```

This simply gives me the output: 

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/gns3lab$ python3 task1.py 
hi! My name is R1 and I live in farringdon
```

<br/>

----

### Working With J2 Templates

https://nornir.readthedocs.io/en/latest/ref/api/nornir.html

#### Nornir Script

Here I have a script that uses napalm to perform a dry run using a jinja template I created:

```
#!/usr/bin/env python3

# Author: Lawrence Long
# generate config and load config (dry_run) on a single device

from nornir import InitNornir
from nornir.plugins.tasks import networking, text
from nornir.plugins.functions.text import print_title, print_result
from nornir.core.filter import F
from nornir.core.exceptions import NornirExecutionError

nr = InitNornir(config_file="config.yaml",
                dry_run=True,
                core={"raise_on_error": True}
                )
dev = nr.filter(F(groups__contains="gns3_firewalls"))


def DryRunConfig(task):
    r = task.run(task=text.template_file,
                 name="Generate configuration",
                 template="vrrp-test.j2",
                 path="templates/")
    task.host["config"] = r.result
    task.run(task=networking.napalm_configure,
             name='Loading configuration on the device',
             replace=False,
             configuration=task.host['config'])


print_title('Playbook to configure the network')
result = dev.run(task=DryRunConfig)
print_result(result)
```

First I give `InitNornir` a new argument, so that these 'changes' be simulated with `dry_run=True` (and argument which defaults to `False`). This argument can also be controlled throught he configuration as well, and some tasks might even allow you to override this behavior at the task level.

I then use `core={"raise_on_error": True}` to override the `raise_on_error` behavior, so that nornir will automatically raise the exception in the case of an error:

```
nr = InitNornir(config_file="config.yaml",
                dry_run=True,
                core={"raise_on_error": True}
                )
```

* https://nornir.readthedocs.io/en/latest/tutorials/intro/failed_tasks.html

Afterwards, I filter my hosts based on the group it belongs to, `gns3_firewalls`.

Because groups is a list, attempting to filter it using the same method as say `platform="junos"` i.e. `groups='gns3_firewalls'` will not yield any hosts and therefore fail. 

Therefore when filtering a `group`, the correct method is: 

```
dev = nr.filter(F(groups__contains="gns3_firewalls"))
```

Then I define a function for my **task** `DryRunConfig`, whereby it will be doing two things. 

1. Render a configuration from the jinja2 template and storing it into a host variable. 
2. Deploying that configuration using NAPALM.

```
def DryRunConfig(task):
    # Transform inventory data to configuration via a template file
    r = task.run(task=text.template_file,
                 name="Generate configuration",
                 template="vrrp-test.j2",
                 path=f"templates/{task.host.platform}")
    # Save the compiled configuration into a host variable                 
    task.host["config"] = r.result

    # Deploy that configuration to the device using NAPALM
    task.run(task=networking.napalm_configure,
             name='Loading configuration on the device',
             replace=False,
             configuration=task.host['config'])
```             

Here the function `DryRunConfig` has one parameter, `task`:

```
def DryRunConfig(task):
```

I can then provide a default value to my function argument using the assignment operator `=` and begin adding the relevant information to render the contents of my jinja2 file.

Using the `text` plugin for `tasks` - `nornir.plugins.tasks.text.template_file` I set the following parameters: 

`template` being the template filename.

`path` is the path to the directory with templates. The 'correct' path can also be manipulated with additional variables such as the `host` `platform`:

```
path=f"templates/{task.host.platform}"
```

* It's important that you format this an as f-string so the string is reprsented correctly as a string literal.
  * More information on f-strings here https://realpython.com/python-f-strings/

This is structred like so:

```
lawrence@ulysses:~/git/moo/nornir$ tree templates/
templates/
├── junos
│   └── vrrp-test.j2
```

* More information on this here https://nornir.readthedocs.io/en/latest/tutorials/intro/grouping_tasks.html

```
    # Transform inventory data to configuration via a template file
    r = task.run(task=text.template_file,
                 name="Generate configuration",
                 template="vrrp-test.j2",
                 path=f"templates/{task.host.platform}")
```

https://nornir.readthedocs.io/en/latest/plugins/tasks/text.html

Then I save the compiled configuration into a host variable `task.host["config"]`:

```
    # Save the compiled configuration into a host variable 
    task.host["config"] = r.result
```    

And lastly to deploy that configuration to the device using NAPALM: 

```
    # Deploy that configuration to the device using NAPALM
    task.run(task=networking.napalm_configure,
             name='Loading configuration on the device',
             replace=False,
             configuration=task.host['config'])
```

The last few lines of code:

```
print_title('Playbook to configure the network')
result = dev.run(task=DryRunConfig)
print_result(result)
```

`text.print_title()` does exactly as it suggests and prints whatever arg I give it when I run the file.

I then create a `result` object whereby I execute the task against my list of hosts defined in `dev = nr.filter(F(groups__contains="gns3_firewalls"))`. 

And finally I use nornir's `text.print_result` function to inspect the results.


<br/>

#### Jinja Template

My jinja template located in `templates/vrrp-test.j2`: 

```
system {
    host-name gns3labvsrx;
}
interfaces {
    ge-0/0/4 {
        unit 0 {
            family inet {
                address {{ host.vrrp_group_1.address }};
            }
        }
    }
}
```

The host variable called in my j2 template `host.vrrp_group_1.address` is represented inside my `hosts.yaml` file as:

```
---
VSRX1:
    hostname: 10.130.3.7
    groups:
        - gns3_firewalls
    platform: junos
    data:
        type: vsrx
        vrrp_group_1: 
            vip: 149.11.141.138
            address: 149.11.141.141/29
            priority: 100
```

#### Output

And my output when ran:

```
lawrence@ulysses:~/git/moo/nornir$ python3 dry_run_autoerror.py                                                                                                          
**** Playbook to configure the network *****************************************
DryRunConfig********************************************************************
* VSRX1 ** changed : True ******************************************************
vvvv DryRunConfig ** changed : False vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv INFO
---- Generate configuration ** changed : False --------------------------------- INFO
system {
    host-name gns3labvsrx;
}
interfaces {
    ge-0/0/4 {
        unit 0 {
            family inet {
                address 149.11.141.141/29;
            }
        }
    }
}
---- Loading configuration on the device ** changed : True --------------------- INFO
[edit system]
-  host-name gns3-vsrx-01;
+  host-name gns3labvsrx;
[edit interfaces]
+   ge-0/0/4 {
+       unit 0 {
+           family inet {
+               address 149.11.141.141/29;
+           }
+       }
+   }
^^^^ END DryRunConfig ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
{}
```

<br/>

---- 

### Using Tasks

https://nornir.readthedocs.io/en/latest/tutorials/intro/executing_tasks.html?highlight=task.host.name#What-is-a-task

OK so what is a task? In it’s simplest form **a task is a function that takes at least a `Task` object as argument**.

#### Task Example 1: 

```
def hi(task):
    print(f"hi! My name is {task.host.name} and I live in {task.host['site']}")

nr.run(task=hi, num_workers=1)
```

#### Output:

```
hi! My name is host1.cmh and I live in cmh
hi! My name is host2.cmh and I live in cmh
```

The task object has access to `nornir`, `host` and `dry_run` attributes. 

#### Task Example 2:

Within this example I call other tasks from within a task such as `networking.napalm_configure` and make the `dry_run` attribute `True`. 

Now I can transform my inventory data to config via a jinja template, compile this configuration into a host variable `task.host["config"]` which I then pass to `napalm_configure` and perform `dry_run=True` all as one task:

```
def napalm_dryrun(task):
    # transform inventory data to config via a template
    r = task.run(task=text.template_file,
                 name="Generate configuration",
                 template=jinjatemplate,
                 path=f'templates/{task.host.platform}')
    # save the compiled configuration into a host variable            
    task.host["config"] = r.result
    # commit-check with napalm
    task.run(task=networking.napalm_configure,
             dry_run=True,
             name='Loading configuration on the device',
             replace=False,
             configuration=task.host['config'])
```

By setting this attribute within the taskk itself I can be specific about what I want to happen on a per task basis rather than having to set this option globally when I initialize nornir:

```
nr = InitNornir(config_file="config.yaml",
                dry_run=True,
                core={"raise_on_error": True}
                )
```                

<br/>

A task is basically a wrapper around a function that has to be run against multiple devices. You won’t probably have to deal with this class yourself as `nornir.core.Nornir.run()` will create it automatically.

This simple function takes two arguments, where task is basically a wrapper around the function and `catbreed` is positional argument. 

The most straightforward way to pass arguments to a Python function is with positional arguments (also called required arguments). 

More information on functions: https://realpython.com/defining-your-own-python-function/#argument-passing

```
def hi(task, catbreed):
    print(f"hi! My name is {task.host.name} and I live in {task.host['site']} " + catbreed)


result = devices.run(task=hi, num_workers=1, catbreed="sphyx_arg")
```

And the output:

```
hi! My name is VSRX01 and I live in lab sphyx_arg
```

----

### Using `transform_function`

You can pass additional arguments such as `connection_options` in your script using the `transform_function`:

```
def adapt_host_data(host):
    # This function receives a Host object for manipulation
    host.username = 'nornir'
    host.password = 'cisco'
    host.connection_options["netmiko"] = ConnectionOptions(
        extras={'use_keys':True}
    )

nr = InitNornir(
    config_file="config.yaml",
    inventory={
        "transform_function": adapt_host_data
    },
    dry_run=True
)
```

More information here: https://nornir.readthedocs.io/en/latest/howto/transforming_inventory_data.html?highlight=host.connection_options#Using-ConnectionOptions


----

To finish...

```
lawrence@ulysses:~/git/Junos/automation/python/nornir/ipvzero$ python3 nornir_script1.py
netmiko_send_command************************************************************
* R1 ** changed : False ********************************************************
vvvv netmiko_send_command ** changed : False vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv INFO
Current time: 2020-02-16 16:18:50 UTC
Time Source:  LOCAL CLOCK 
System booted: 2020-02-16 12:21:17 UTC (03:57:33 ago)
Protocols started: 2020-02-16 12:26:13 UTC (03:52:37 ago)
Last configured: 2020-02-16 16:05:16 UTC (00:13:34 ago) by lawrence
 4:18PM  up 3:58, 11 users, load averages: 0.67, 0.57, 0.56

^^^^ END netmiko_send_command ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

----- 

#### Documentation
* https://nornir.discourse.group/t/using-keybasd-ssh-to-login-to-a-remote-linux-box/106
