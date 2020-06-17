---
layout: post
title: Passing arguments in BASH
description: Passing arguments in BASH
summary: Passing arguments in BASH
comments: false
---

I recently wrote this for a friend at work who was trying to pass arguments to a BASH script he was writing. Here I will go over how you can pass arguments such as string, files and even flags to a shell script.

<br/>

### Passing arguments

Lets say I have a script whereby I need to pass a file name as an argument so the script can perform an action with said file. 

Freely passing arguments in this manner can be extremely effective as it prevents us from having to hardcode the arguments. Maybe you're working with multiple CSV files, or trying to pass the name of a running program to an aux script. 

Arguments are passed via the cli and are accessed as variables `$1`, `$2`, `$3` etc, with the number denoting the argument given:

```
lawrence@cacti:~/git/Junos/bash$ cat aux_awk.sh 
#!/bin/bash

FILE1=$1
FILE2=$2
cat $FILE1
cat $FILE2
```

```
lawrence@cacti:~/git/Junos/bash$ bash aux_awk.sh helloworld.txt goodbyeworld.txt 
Hello world
Goodbye world
```

If however, I am working with an arbitrary number of args, I can use the `$@` variable and access these from within an array:

```bash
FILES=$@
cat $FILES
```

```
lawrence@cacti:~/git/Junos/bash$ bash aux_awk.sh helloworld.txt goodbyeworld.txt 
Hello world
Goodbye world
```

This is also the same as creating a `for` loop:

```bash
for FILE in "$@"
do
    cat $FILE
done
```

As you can see the results are the same:

```
lawrence@cacti:~/git/Junos/bash$ bash aux_awk.sh helloworld.txt goodbyeworld.txt 
Hello world
Goodbye world
```

It doesn't just have to be a file, the same can be done with strings:

```bash
NAME=$@
ps aux | grep $NAME | awk '{print $2}'
```

This grabs the `PID` of the process name given:

```
lawrence@cacti:~/git/Junos/bash$ bash aux_awk.sh quake
17815
17817
```

And for multiple programs:

```bash
#!/bin/bash
NAME=$@
for N in $NAME
do 
    ps aux | grep $N #| awk '{print $2}'
done
```

```
lawrence@cacti:~/git/Junos/bash$ bash aux_awk.sh quake teamviewer
lawrence 22756  0.0  0.0  19988  3288 pts/4    S+   20:44   0:00 bash aux_awk.sh quake teamviewer
lawrence 22758  0.0  0.0  21532  1052 pts/4    S+   20:44   0:00 grep quake
root      1843  0.1  0.2 1553652 41068 ?       Sl   Jun12   7:38 /opt/teamviewer/tv_bin/teamviewerd -d
lawrence 22756  0.0  0.0  19988  3288 pts/4    S+   20:44   0:00 bash aux_awk.sh quake teamviewer
lawrence 22760  0.0  0.0  21532  1036 pts/4    S+   20:44   0:00 grep teamviewer
```

<br/>

### Passing flags

```bash
#!/bin/bash
while getopts u:d:p:f: option
do
    case "${option}"
in
    u) UNAME=${OPTARG};;
    d) FNAME=${OPTARG};;
esac
done

echo $UNAME
echo $FNAME
```

```
lawrence@cacti:~/git/Junos/bash$ bash passing_flags.sh -u lawrence -d 'Lawrence Long'
lawrence
Lawrence Long
```
