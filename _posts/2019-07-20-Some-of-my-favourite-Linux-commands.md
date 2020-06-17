---
layout: post
title: Some of my favourite Linux commands
description: Some of my favourite Linux commands
summary: Some of my favourite Linux commands
comments: false
---

The Linux kernal and other components are free and open-source, this means code is available to the public to view, edit, and for those that wish to contribute - may do so. Here I will go through some of the more widely used commands that I have used as a systems engineer working within DevOps.

<br/>

### Tree

One of my favourite commands is `tree`. It allows you to recursively show the contents of a directory but in a tree-like format. I find it can be particularly useful to be able to print out structures in this manner when doing workshops with colleagues. The only caveat to `ls` is that it is not always installed by default, therefore I highly recommend adding it to your builds.

Whilst it is true you can recursively list the contents of directories with `ls -R`, I personally feel like this:

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ ls -R aliens/
aliens/:
aliens-task.yml  group_vars

aliens/group_vars:
all.yml
```

Is less readable than using `tree` to effectively achieve the same result:

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ tree aliens/
aliens/
├── aliens-task.yml
└── group_vars
    └── all.yml
```

Hidden files can be enabled using the `-a` flag.

And you can prevent any files from being shown, instead printing only the subdirectories with the `-d` flag: 

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ tree -d aliens/
aliens/
└── group_vars
```

You can even print out the username and group, along with the file type and permissions with `-pug`, which is the equivalent of doing `ls -l`:

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ tree -pug aliens/
aliens/
├── [-rw-rw-r-- lawrence lawrence]  aliens-task.yml
└── [drwxrwxr-x lawrence lawrence]  group_vars
    └── [-rw------- lawrence lawrence]  all.yml
```

The file size can be viewed with `-sh` (the `-h` flag makes the size output 'human' friendly):

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ tree -sh aliens/
aliens/
├── [ 619]  aliens-task.yml
└── [4.0K]  group_vars
    └── [  88]  all.yml
```    

This could alternatively be achieved with `ls -lh`:

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ ls -R -ls aliens
aliens:
total 8
4 -rw-rw-r-- 1 lawrence lawrence  619 Jun 11 11:22 aliens-task.yml
4 drwxrwxr-x 2 lawrence lawrence 4096 Jun 11 13:02 group_vars

aliens/group_vars:
total 4
4 -rw------- 1 lawrence lawrence 88 Jun 11 13:01 all.yml
```

Lastly you can **match** a pattern with with the `-P` flag.

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ tree -pug aliens/ -P all*
aliens/
└── [drwxrwxr-x lawrence lawrence]  group_vars
    └── [-rw------- lawrence lawrence]  all.yml
```    

To use `ls` you could achieve something similar to this by passing the output of `ls` to `grep`:

```
lawrence@cacti:~/git/Ansible/vmware/playbooks$ ls -R aliens/ | grep all.*
all.yml
```

Hopefully some of these examples have shown you just how awesome `tree` is and has inspired you to add it to your daily toolkit.

<br/>

### Cat

`cat` which is short for 'concatenate' is a command I find myself regularly using, along with `less` which is more preferable for viewing longer files (no one wants their terminal drowned in file output).

Did you know that you can create a file with `cat`? By pre-pending the desired filename with a redirection `>`, it will open an interactive shell through which you can begin entering text (simply quit out of it afterwards with `CTRL` `D` or `CTRL` `C`):

```
lawrence@cacti:~$ cat >helloworld.txt
hello world!
lawrence@cacti:~$ cat helloworld.txt 
hello world!
```

You can even redirect the output of multiple files to a single file like so:

```
lawrence@cacti:~$ cat helloworld.txt goodbyeworld.txt >hellogoodbye.txt
lawrence@cacti:~$ cat hellogoodbye.txt 
hello world!
goodbye world
```

If you're working with large file output, you can pipe to `less`, or you can simply omit `cat` completely and do `less helloworld.txt`:

```
lawrence@cacti:~$ cat helloworld.txt | less

hello world!
(END)
```

Lastly, you can list out the line numbers of a file with the `-n` flag. You can even pass this to `less` which I've found to be very useful:

```
lawrence@cacti:~$ cat helloworld.txt -n | less

     1  hello world!
(END)
```

<br/>

### Grep

`grep` allows you to search for a regular expression, such as a word or a string, within a plain-text data set. Of the `grep` tools that exist, 

`fgrep` is for fixed strings in a file and is the same as doing `grep -F`. It is useful for when you need to search for strings which contain lots of regular expressions metacharacters:

```
lawrence@cacti:~$ cat words.txt 
h\ello
wor.ld
good*bye
```

```
lawrence@cacti:~$ cat hellogoodbye.txt 
h\ello 
wor.ld! 
good*bye 
world
```

Here, `grep` will fail to find all the words (the `-f` flag or `--file` allows me to provide a text file):

```diff
lawrence@cacti:~$ grep -f words.txt hellogoodbye.txt
! wor.ld!
```

But with `fgrep` all the words are successfully matched:

```diff
lawrence@cacti:~$ fgrep -f words.txt hellogoodbye.txt
+ h\ello 
  wor.ld! 
+ good*bye 
```

Another cool feature of `grep` is that you can do an inverse match with `-v` or `--invert-match`:

```
lawrence@cacti:~$ cat somecode.sh 
#!/bin/bash
echo "Hello World"
# do something else
echo $PATH

lawrence@cacti:~$ grep -v '#' somecode.sh 
echo "Hello World"
echo $PATH
```

This is particuarly useful when you you're trying to omit commented out lines from code.

You can even search directories recursively with the `-r` flag:

```
lawrence@cacti:~$ grep -r 'Hello World' bash/ 
bash/somecode.sh:echo "Hello World"
```

Last but not least, probably one of the most common examples is to pipe the output of one program to `grep` to perform matching with. For example `ls` does not do pattern matching:

```
lawrence@cacti:~$ ls -al dir/
total 16
drwxrwxr-x  2 lawrence lawrence 4096 Jun 16 18:51 .
drwxr-xr-x 47 lawrence lawrence 4096 Jun 16 18:51 ..
-rw-rw-r--  1 lawrence lawrence   33 Jun 16 17:45 hellogoodbye.txt
-rw-rw-r--  1 lawrence lawrence   13 Jun 16 15:44 helloworld.txt
```

However when combined with `grep` it can be done:

```
lawrence@cacti:~$ ls -al dir/ | grep world
-rw-rw-r--  1 lawrence lawrence   13 Jun 16 15:44 helloworld.txt
```

<br/>

### `chown` and `chmod`

I've combined these two because I find myself regularly using them together. 

`chown` allows you to change the ownership of a file or directory, this can encompass the **owner** and **group**. 

```
lawrence@cacti:~$ sudo chown nobody:nogroup dir/helloworld.txt 
lawrence@cacti:~$ ls -al dir/ | grep world
-rw-rw-r--  1 nobody   nogroup    13 Jun 16 15:44 helloworld.txt
```

`chmod` on the other hand, is used to change the access permissions such as read-write and read-only, and can be applied for the **owner**, **group** and **everyone**.

* A really awesome tool worth checking out is https://chmod-calculator.com/ (this regularly saves me a lot of hassle!).

```
lawrence@cacti:~$ sudo chmod 440 dir/helloworld.txt 
lawrence@cacti:~$ cat dir/helloworld.txt 
cat: dir/helloworld.txt: Permission denied
```

<br/>

### Lsof

The `lsof` command stands for **list open files** and can be used to see which files are open by a particular process, this includes **pipes**, **sockets**, **directories**, and **devices** etc (all considered by Unix as files).

A scenario where you might use this is if you're unable to unmount a disk as the files are apparently in use, with `lsof` you could see which files are responsible. Or perhaps you're trying to find the processes running on a specific port.

You can list opened files for a particular **user** with the `-u` flag:

```
lawrence@cacti:~$ lsof -u lawrence | tail -n 10
lsof: WARNING: can't stat() tracefs file system /sys/kernel/debug/tracing
      Output information may be incomplete.
bash      27138 lawrence  mem       REG                8,2     14560    2364156 /lib/x86_64-linux-gnu/libdl-2.27.so
bash      27138 lawrence  mem       REG                8,2    170784    2364291 /lib/x86_64-linux-gnu/libtinfo.so.5.9
bash      27138 lawrence  mem       REG                8,2    170960    2364105 /lib/x86_64-linux-gnu/ld-2.27.so
bash      27138 lawrence  mem       REG                8,2      3416    8396159 /usr/share/locale-langpack/en_GB/LC_MESSAGES/libc.mo
bash      27138 lawrence  mem       REG                8,2     23665    8398194 /usr/share/locale-langpack/en_GB/LC_MESSAGES/bash.mo
bash      27138 lawrence  mem       REG                8,2     26376    7474039 /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
bash      27138 lawrence    0u      CHR             136,10       0t0         13 /dev/pts/10
bash      27138 lawrence    1u      CHR             136,10       0t0         13 /dev/pts/10
bash      27138 lawrence    2u      CHR             136,10       0t0         13 /dev/pts/10
bash      27138 lawrence  255u      CHR             136,10       0t0         13 /dev/pts/10
```

With the `-i` flag you can select 'the listing of files any of whose Internet address matches the address specified in i'.

Here I'm able to specify the **Protocol** and **Port** number to identify the process:

```
lawrence@cacti:~$ lsof -i TCP:22
COMMAND  PID     USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
ssh     5622 lawrence    3u  IPv4 2316553      0t0  TCP cacti:53852->XXX.XX.XX.XXX:ssh (ESTABLISHED)
```

You can even show network files only for IPv4 and IPv6 with the `-i 4` and `-i 6` args:

```
lawrence@cacti:~$ lsof -i 4 | head -5
COMMAND     PID     USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
chrome     5617 lawrence  173u  IPv4 1887457      0t0  UDP 224.0.0.251:mdns 
chrome     5658 lawrence   31u  IPv4 2312250      0t0  TCP cacti:39492->lb-140-82-114-25-iad.github.com:https (ESTABLISHED)
chrome     5658 lawrence   37u  IPv4 2193958      0t0  TCP cacti:53124->a2-21-184-19.deploy.static.akamaitechnologies.com:https (ESTABLISHED)
chrome     5658 lawrence   40u  IPv4 2130402      0t0  TCP cacti:50142->a23-36-212-221.deploy.static.akamaitechnologies.com:https (ESTABLISHED)
```


