---
layout: post
title: Filtering Output In Ansible
description: Filtering Output In Ansible
summary: Filtering Output In Ansible
comments: false
---

Recently I've been teaching a couple of my colleagues how to use Ansible, and one of the questions they had was how to work with task output, particularly so when it's not a dictionary. 

Welcome to [Return Values](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html). To reference the official documentation, Ansible modules return a data structure that can be registered into a variable, or seen directly when output by a playbook. 

Take for example, the [juniper_junos_facts](https://junos-ansible-modules.readthedocs.io/en/2.0.0/juniper_junos_facts.html) module, which collects fact information from a remote Junos device using PyEZ, and returns the results within a **dictionary**:

```yaml
{% raw %}        
  tasks:
    - name: save device configuration
      juniper_junos_facts:
        provider: "{{ connection_settings }}"
      register: junos_facts

    - name: show junos_facts
      debug:
        var: junos_facts      
{% endraw %}        
```

For each device, `juniper_junos_facts` returns a dictionary which I then **register** as `junos_facts`. Within my returned `junos_facts` dictionary, are two keys that appear to hold the same information; `ansible_facts` and `facts`:

```json
{% raw %}        
TASK [show junos_facts] **************************************************************************************************************************************************************************************************
ok: [breen] => {
    "junos_facts": {
        "ansible_facts": {
            "junos": {
                "HOME": "/var/home/lawrence",
                "RE0": null,
                "RE1": null,
                "RE_hw_mi": false,
                "current_re": [
                    "master",
                    "node",
                    "fwdd",
                    "member",
                    "pfem",
                    "re0",
                    "fpc0",
                    "localre"
                ],
                "domain": null,
                "fqdn": "vqfx-re",
                "has_2RE": false,
                "hostname": "vqfx-re",
                "hostname_info": {
                    "fpc0": "vqfx-re"
                },
                "ifd_style": "CLASSIC",
                "junos_info": {
                    "fpc0": {
                        "object": {
                            "build": 10,
                            "major": [
                                19,
                                4
                            ],
                            "minor": "1",
                            "type": "R"
                        },
                        "text": "19.4R1.10"
                    }
                },
                "master": null,
                "master_state": true,
                "model": "VQFX-10000",
                "model_info": {
                    "fpc0": "VQFX-10000"
                },
                "personality": null,
                "re_info": null,
                "re_master": null,
                "re_name": "fpc0",
                "serialnumber": "VM5E2567B514",
                "srx_cluster": null,
                "srx_cluster_id": null,
                "srx_cluster_redundancy_group": null,
                "switch_style": "VLAN_L2NG",
                "vc_capable": true,
                "vc_fabric": false,
                "vc_master": "0",
                "vc_mode": "Enabled",
                "version": "19.4R1.10",
                "version_RE0": null,
                "version_RE1": null,
                "version_info": {
                    "build": 10,
                    "major": [
                        19,
                        4
                    ],
                    "minor": "1",
                    "type": "R"
                },
                "virtual": null
            }
        },
        "changed": false,
        "facts": {
            "HOME": "/var/home/lawrence",
            "RE0": null,
            "RE1": null,
            "RE_hw_mi": false,
            "current_re": [
                "master",
                "node",
                "fwdd",
                "member",
                "pfem",
                "re0",
                "fpc0",
                "localre"
            ],
            "domain": null,
            "fqdn": "vqfx-re",
            "has_2RE": false,
            "hostname": "vqfx-re",
            "hostname_info": {
                "fpc0": "vqfx-re"
            },
            "ifd_style": "CLASSIC",
            "junos_info": {
                "fpc0": {
                    "object": {
                        "build": 10,
                        "major": [
                            19,
                            4
                        ],
                        "minor": "1",
                        "type": "R"
                    },
                    "text": "19.4R1.10"
                }
            },
            "master": null,
            "master_state": true,
            "model": "VQFX-10000",
            "model_info": {
                "fpc0": "VQFX-10000"
            },
            "personality": null,
            "re_info": null,
            "re_master": null,
            "re_name": "fpc0",
            "serialnumber": "VM5E2567B514",
            "srx_cluster": null,
            "srx_cluster_id": null,
            "srx_cluster_redundancy_group": null,
            "switch_style": "VLAN_L2NG",
            "vc_capable": true,
            "vc_fabric": false,
            "vc_master": "0",
            "vc_mode": "Enabled",
            "version": "19.4R1.10",
            "version_RE0": null,
            "version_RE1": null,
            "version_info": {
                "build": 10,
                "major": [
                    19,
                    4
                ],
                "minor": "1",
                "type": "R"
            },
            "virtual": null
        },
        "failed": false,
        "warnings": [
            "The value 2222 (type int) in a string field was converted to '2222' (type string). If this does not look like what you expect, quote the entire value to ensure it does not change."
        ]
    }
}      
{% endraw %}
```

The nested `junos_facts.ansible_facts.junos` key contains facts returned by our `juniper_junos_facts` module, however, the `junos_facts.facts` key duplicates these same facts, primarily for backwards compatibility. I can reduce the output by calling only one of the nested keys, preventing the facts from being duplicated:

```yaml
    - name: show junos_facts.ansible_facts.junos
      debug:
        var: junos_facts.ansible_facts.junos
```        

Now when I run the playbook, I'll receive only one set of facts, specifically those stored in `junos_facts.ansible_facts.junos`:

```json
{% raw %}        
TASK [show junos_facts.ansible_facts.junos] ******************************************************************************************************************************************************************************
ok: [breen] => {
    "junos_facts.ansible_facts.junos": {
        "HOME": "/var/home/lawrence",
        "RE0": null,
        "RE1": null,
        "RE_hw_mi": false,
        "current_re": [
            "master",
            "node",
            "fwdd",
            "member",
            "pfem",
            "re0",
            "fpc0",
            "localre"
        ],
        "domain": null,
        "fqdn": "vqfx-re",
        "has_2RE": false,
        "hostname": "vqfx-re",
        "hostname_info": {
            "fpc0": "vqfx-re"
        },
        "ifd_style": "CLASSIC",
        "junos_info": {
            "fpc0": {
                "object": {
                    "build": 10,
                    "major": [
                        19,
                        4
                    ],
                    "minor": "1",
                    "type": "R"
                },
                "text": "19.4R1.10"
            }
        },
        "master": null,
        "master_state": true,
        "model": "VQFX-10000",
        "model_info": {
            "fpc0": "VQFX-10000"
        },
        "personality": null,
        "re_info": null,
        "re_master": null,
        "re_name": "fpc0",
        "serialnumber": "VM5E2567B514",
        "srx_cluster": null,
        "srx_cluster_id": null,
        "srx_cluster_redundancy_group": null,
        "switch_style": "VLAN_L2NG",
        "vc_capable": true,
        "vc_fabric": false,
        "vc_master": "0",
        "vc_mode": "Enabled",
        "version": "19.4R1.10",
        "version_RE0": null,
        "version_RE1": null,
        "version_info": {
            "build": 10,
            "major": [
                19,
                4
            ],
            "minor": "1",
            "type": "R"
        },
        "virtual": null
    }
}      
{% endraw %}
```

Now if I wanted to filter this output further and access the `hostname` value, I simply need append my dictionary lookup with the `.hostname` key:

```yaml
    - name: Get hostname
      debug: 
        var: junos_facts.ansible_facts.junos.hostname
```

This way, I am able to display specific data about my network device and create my own facts based on these key-value pairs by calling them from within my dictionary.

There is however, a simpler way to access these attributes, and it involves using the [ansible_facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html) **dictionary**. 

A nice feature of Ansible is that it stores any host facts it discovers within an `ansible_facts` **dictionary**. Furthermore, the `juniper_junos_facts` module automatically registers these facts to the `ansible_facts` dictionary under a `junos` key.

Meaning that I need not register my dictionary in a variable with `register: junos_facts`. Instead I can skip the `register` part entirely and simply call the `junos` dictionary like so:

```yaml
    - name: show hostname
      debug:
        var: junos.hostname
```

See how much simpler it is to use `junos.hostname` instead of `junos_facts.ansible_facts.junos.hostname`, and yet the output is identical:

```json
{% raw %}        
TASK [show hostname] **************************************************************************************************************************************************************************
ok: [breen] => {
    "junos.hostname": "vqfx-re"
}     
{% endraw %}
```

<br/>

### Filtering output using common return values

You'll find that most vendor modules are capable of returning data structures within a dictionary, with many offering options such as JSON or YAML, and even `ansible_facts` making them directly accessible without having to register as a variable. 

There will however, be times that you need to work with output that isn't contained within a dictionary and this is where things get a little bit tricky, as you cannot simply access the key of the desired attribute, instead it may be a list of strings which requires filtering to achieve a certain value. 

For example, lets use the `command` module and get some CPU information about my machine using `lscpu`:

```yaml
{% raw %}
# Gets information from output using split() and lists
---
- name: Get CPU info
  hosts: localhost
  gather_facts: no

  tasks:
    - name: Get CPU information
      command: lscpu
      register: lscpu_info

    - name: Return CPU information
      debug:
        msg: "{{ lscpu_info }}"
{% endraw %}
```        

Now lets run the playbook and look at the output stored in our `{{ lscpu_info }}` variable:

```json
{% raw %}        
TASK [Return all CPU Information as a list of strings] ****************************************************************************************************************************************
ok: [localhost] => {
    "msg": {
        "changed": true,
        "cmd": [
            "lscpu"
        ],
        "delta": "0:00:00.022884",
        "end": "2020-07-20 14:28:45.387714",
        "failed": false,
        "rc": 0,
        "start": "2020-07-20 14:28:45.364830",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "Architecture:        x86_64\nCPU op-mode(s):      32-bit, 64-bit\nByte Order:          Little Endian\nCPU(s):              8\nOn-line CPU(s) list: 0-7\nThread(s) per core:  2\nCore(s) per socket:  4\nSocket(s):           1\nNUMA node(s):        1\nVendor ID:           GenuineIntel\nCPU family:          6\nModel:               60\nModel name:          Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz\nStepping:            3\nCPU MHz:             3299.312\nCPU max MHz:         4400.0000\nCPU min MHz:         800.0000\nBogoMIPS:            8000.03\nVirtualisation:      VT-x\nL1d cache:           32K\nL1i cache:           32K\nL2 cache:            256K\nL3 cache:            8192K\nNUMA node0 CPU(s):   0-7\nFlags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts md_clear flush_l1d",
        "stdout_lines": [
            "Architecture:        x86_64",
            "CPU op-mode(s):      32-bit, 64-bit",
            "Byte Order:          Little Endian",
            "CPU(s):              8",
            "On-line CPU(s) list: 0-7",
            "Thread(s) per core:  2",
            "Core(s) per socket:  4",
            "Socket(s):           1",
            "NUMA node(s):        1",
            "Vendor ID:           GenuineIntel",
            "CPU family:          6",
            "Model:               60",
            "Model name:          Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz",
            "Stepping:            3",
            "CPU MHz:             3299.312",
            "CPU max MHz:         4400.0000",
            "CPU min MHz:         800.0000",
            "BogoMIPS:            8000.03",
            "Virtualisation:      VT-x",
            "L1d cache:           32K",
            "L1i cache:           32K",
            "L2 cache:            256K",
            "L3 cache:            8192K",
            "NUMA node0 CPU(s):   0-7",
            "Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts md_clear flush_l1d"
        ]
    }
}     
{% endraw %}
```

Yikes, this is definitely a far cry from what we were working with previously when each attribute was conveniently nested within a key inside our dictionary.

Yet again, we have duplicate output, however in this case, we have a string literal inside `"stdout":` which has our facts, but also includes line feed characters `\n` used to indicate a new line. And inside `"stdout_lines":` we have our duplicated facts, but no visible line feed characters, instead our facts are presented in a readable manner on each line.

As we're going to be working with string output, it is therefore in our interest that we access the `.stdout_lines` key, which will always provide a list of strings, each containing one item per line from the original output. This is referred to as a **common return value**.

You can actually get a better view of the data structure by pasting everything enclosed within our `{ }` (including the braces) into a JSON viewer such as [JSONviewer.stack.hu](http://jsonviewer.stack.hu/).

<br/>

### Working with stdout_lines

Now that we know how we're going to return our output, we need to figure out how we're going to turn our strings into meaningful values that we can actually work with. 

First let's take another look at our output, this time using `lscpu_info.stdout_lines`:

```json
{% raw %}        
TASK [Return all CPU Information as a list of strings] ****************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "Architecture:        x86_64",
        "CPU op-mode(s):      32-bit, 64-bit",
        "Byte Order:          Little Endian",
        "CPU(s):              8",
        "On-line CPU(s) list: 0-7",
        "Thread(s) per core:  2",
        "Core(s) per socket:  4",
        "Socket(s):           1",
        "NUMA node(s):        1",
        "Vendor ID:           GenuineIntel",
        "CPU family:          6",
        "Model:               60",
        "Model name:          Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz",
        "Stepping:            3",
        "CPU MHz:             3299.312",
        "CPU max MHz:         4400.0000",
        "CPU min MHz:         800.0000",
        "BogoMIPS:            8000.03",
        "Virtualisation:      VT-x",
        "L1d cache:           32K",
        "L1i cache:           32K",
        "L2 cache:            256K",
        "L3 cache:            8192K",
        "NUMA node0 CPU(s):   0-7",
        "Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts md_clear flush_l1d"
    ]
}   
{% endraw %}
```

Now the key thing to note here, is that this is a **list of strings**, which means we should in fact be able to access each string (or line) of our output by calling its corresponding index:

```yaml
{% raw %}
    - name: Return first line in my list with index as key; 'Architecture:'
      debug:
        msg: "{{ lscpu_info.stdout_lines[0] }}"
{% endraw %}        
```

By calling the index `[0]` I can get the first element from our list, in this case, the `Architecture` of our CPU:

```json
{% raw %}        
ok: [localhost] => {
    "msg": "Architecture:        x86_64"
}     
{% endraw %}
```

So now that I have my `Archiecture` string, I can use the Python `.split()` method to split my string into a list of elements which by default, use whitespace as a delimiter:

```yaml
{% raw %}        
    - name: Use split() method on strings at index 0 and 9
      debug:
        msg: 
          - "{{ lscpu_info.stdout_lines[0].split() }}"
          - "{{ lscpu_info.stdout_lines[9].split() }}"          
{% endraw %}          
```          

This provides me with the following strings in list format:

```json
{% raw %}        
ok: [localhost] => {
    "msg": [
        [
            "Architecture:",
            "x86_64"
        ],
        [
            "Vendor",
            "ID:",
            "GenuineIntel"
        ]
    ]
}    
{% endraw %}
```

Now I can store my lists as facts:

```yaml
{% raw %}        
    - name: Set lscpu facts; arch_cpu_var, vendor_cpu_var, mhz_cpu_var
      set_fact: 
        arch_cpu_var: "{{ lscpu_info.stdout_lines[0].split() }}"
        opmode_cpu_var: "{{ lscpu_info.stdout_lines[1].split() }}"
        vendor_cpu_var: "{{ lscpu_info.stdout_lines[9].split() }}"
        mhz_cpu_var: "{{ lscpu_info.stdout_lines[14] }}"   
{% endraw %}
```

And to access the elements containing the values I need, I call the index of the element I want, just as I did originally with `.stdout_lines`:

```yaml
{% raw %}        
    - name: Print facts by calling the index of my stored lists
      debug:
        msg: 
          - "The Architecture of my CPU is {{ arch_cpu_var[1] }}" 
          - "The Vendor ID of my CPU is {{ vendor_cpu_var[2] }}"     
{% endraw %}          
```          

And that's how you can create variables from strings using the `.split()` method:

```json
{% raw %}        
TASK [Print facts by calling the index of my stored lists] ************************************************************************************************************************************
ok: [localhost] => {
    "msg": [
        "The Architecture of my CPU is x86_64",
        "The Vendor ID of my CPU is GenuineIntel"
    ]
}     
{% endraw %}
```

In the case of `mhz_cpu_var:`, I did not use `split()`, instead I only stored my fact as a string and not a list:

```yaml
{% raw %}        
        mhz_cpu_var: "{{ lscpu_info.stdout_lines[14] }}"
{% endraw %}
```        

I did this purposely so I could continue to work with my output by applying some basic regex like so:

```yaml
{% raw %}        
    # regex - lets extract the decimal number from our current "CPU MHz: 3060.525"
    - name: Extract integer and float from fact using regex_search; mhz_cpu_var
      debug:
        # gets digits, however only gets an integer'3060' with REGEX '\d+'
        msg: 
          - "This is my mhz_cpu_var list: {{ mhz_cpu_var }}"
          - "I extracted the following integer: {{ mhz_cpu_var 
              | regex_search('\\d+') }}"
          - "I extracted the following float: {{ mhz_cpu_var 
              | regex_search('\\d+'+'\\D+'+'\\d+') }}"   
{% endraw %}              
```              

Regex will not work on a list, instead it requires a string to parse. Therefore by combining regex with my string data, I can filter my string for the float value of my current CPU MHz:

```json
{% raw %}
ok: [localhost] => {
    "msg": [
        "This is my mhz_cpu_var list: CPU MHz:             3138.574",
        "I extracted the following integer: 3138",
        "I extracted the following float: 3138.574"
    ]
}
{% endraw %}  
```

Pretty neat huh.

<br/>

### Accessing a list inside a list

Sometimes you will get output in the following format

```json
{% raw %}
    "msg": [
        [
            "uptime: 2w4d1h52m10s",
            "                  version: 6.45.5 (stable)",
            "               build-time: Aug/26/2019 10:56:37",
            "         factory-software: 6.15",
            "              free-memory: 38.8MiB",
            "             total-memory: 64.0MiB",
            "                      cpu: MIPS 74Kc V4.12",
            "                cpu-count: 1",
            "            cpu-frequency: 600MHz",
            "                 cpu-load: 1%",
            "           free-hdd-space: 109.2MiB",
            "          total-hdd-space: 128.0MiB",
            "  write-sect-since-reboot: 40859",
            "         write-sect-total: 49119",
            "               bad-blocks: 0%",
            "        architecture-name: mipsbe",
            "               board-name: RB2011iL",
            "                 platform: MikroTik"
        ]
    ]
{% endraw %}      
```

Notice the two `[ [` braces, this means you'll need to access the **first** list before you can access the **second** list nested within. You can do this just like you would in Python, by simply calling both indexes:

```
{% raw %}
set_fact: 
        uptime_var: "{{ test_output.stdout_lines[0][0].split() }}"
{% endraw %}          
```

The first `[0]` index calls the first element of the list, referring to the list nested inside, hence the second index call `[0][0]` which calls the uptime string `"uptime: 2w4d1h52m10s"`.

<br/>

### Using regular expressions (regex)

Now, there is a caveat of using the `stdout_lines` **index** method, and that is that you need to be sure that your output isn't going to change in some significant way. What I mean is, that you need to be certain that **index** you're referring to, is always going to be there and not replaced with something unexpected. The above method worked great, because I know for certain that the output of `lscpu` is always going to be in the same format; `Architecture` will always be index `[0]` and so on.

If however your output is subject to change, and you cannot guarantee the exact format of your data structure, then relying on the index method becomes no-longer feasible.

This is where regex comes in. 

Before I can start working with my `lscpu` output, I first need to convert my **list of strings** into **one string** so that all of my data can be read by one regex statement:

```yaml
{% raw %}
    - name: Use join() method to create one long string
      debug:
        # Joins list output into one long STRING with 'join(' ')'
        msg: "{{ lscpu_info.stdout_lines 
              | join(' ') }}"
{% endraw %}                
```

By using the `.join()` method I get list of strings from `.stdout_lines` returned in one packaged string:

```json
{% raw %}
ok: [localhost] => {
    "msg": "Architecture:        x86_64 CPU op-mode(s):      32-bit, 64-bit Byte Order:          Little Endian CPU(s):              8 On-line CPU(s) list: 0-7 Thread(s) per core:  2 Core(s) per socket:  4 Socket(s):           1 NUMA node(s):        1 Vendor ID:           GenuineIntel CPU family:          6 Model:               60 Model name:          Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz Stepping:            3 CPU MHz:             3138.574 CPU max MHz:         4400.0000 CPU min MHz:         800.0000 BogoMIPS:            8000.03 Virtualisation:      VT-x L1d cache:           32K L1i cache:           32K L2 cache:            256K L3 cache:            8192K NUMA node0 CPU(s):   0-7 Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts md_clear flush_l1d"
}
{% endraw %}  
```

Now am I ready to begin working with my string data, so let's attempt something a little harder and grab the contents of `Model name:` using only regex:

```yaml
{% raw %}
    - name: Use regex_search on string to get substring
      debug:
        # Model name:          Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz Stepping:
        msg: "{{ lscpu_info.stdout_lines 
              | join(' ') 
              | regex_search('(Model name:.*Stepping:)') }}"
{% endraw %}                
```    

Here I've used `regex_search` to get the **substring** of my string, using `Model name:` as a start and `Stepping:` as an end:

```json
{% raw %}
ok: [localhost] => {
    "msg": "Model name:          Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz Stepping:"
}
{% endraw %}  
```

However I still need to remove `Model name:`, `Stepping:`, and all the whitespace in between to get the information I want. 

To achieve this I'm going to use `regex_replace` to search and replace everything in my string up until `Model name:` with nothing `''`. Afterwhich I will **pipe** the output of my first replace to another regex query but this time, removing everything from `Stepping:` onwards, leaving only the information I want in-between:

```yaml
{% raw %}
    - name: Trim substring with regex_replace and extract model name info
      debug:
        msg: "{{ lscpu_info.stdout_lines 
              | join(' ') 
              | regex_replace('(.*Model name:\\s*)', '') 
              | regex_replace('(\\s*Stepping:.*)', '') }}"
{% endraw %}                
``` 

And now I get the exactly the information I wanted within my substring:

```
{% raw %}
ok: [localhost] => {
    "msg": "Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz"
}
{% endraw %}  
```

Now I can store this inside a fact with `set_facts` and simply use it as a variable in the rest of my playbook if I wish to do so:

```yaml
{% raw %}
    - name: Set fact model name information
      set_fact: 
        model_name: "{{ lscpu_info.stdout_lines 
                      | join(' ') 
                      | regex_replace('(.*Model name:\\s*)', '') 
                      | regex_replace('(\\s*Stepping:.*)', '') }}" 

    - name: Print model_name var
      debug:
        var: model_name
{% endraw %}  
```        

<br/>

Hopefully that gives you an insight into how you can better work with different output types in Ansible =)



