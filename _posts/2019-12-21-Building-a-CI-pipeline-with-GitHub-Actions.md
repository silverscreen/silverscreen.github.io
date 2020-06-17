---
layout: post
title: Building a CI pipeline with GitHub Actions
description: Building a CI pipeline with GitHub Actions
summary: Building a CI pipeline with GitHub Actions
comments: false
---

GitHub offers hosted virtual machines to run workflows, with a vm containing an environment of tools, packages, and settings available for GitHub Actions to use.

When you use a GitHub-hosted runner, machine maintenance and upgrades are taken care of for you. You can run workflows directly on the virtual machine or in a Docker container.

----

### Introduction to Actions

First I create my workflow within the root directory of my repo, within `.github/workflows`.

This file can be whatever you want to call it: 

```
lawrence@ulysses:~/git$ tree lemons/.github                                                         
lemons/.github
└── workflows
    └── blank.yml

```

**The contents of my workflow must follow the workflow syntax.**

* https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions

```
lawrence@ulysses:~/git$ cat lemons/.github/workflows/blank.yml
name: remote ssh command
on: [push]
jobs:

  build:
    name: Build
    runs-on: ubuntu-latest
    steps:
    - name: executing remote ssh commands using password
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.PRIV_KEY }}
        port: ${{ secrets.PORT }}
        script: touch createdbygithubaction.txt
```

Here I'm using https://github.com/appleboy/ssh-action for executing remote ssh connections to my linux EC2 instance in AWS. 

Upon committing, it will execute commands given to `script`, which in this case creates a .txt file in the home user directory.

Now I'm ready to push any changes to my repo: 

```
lawrence@ulysses:~/git/lemons$ git push
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 424 bytes | 424.00 KiB/s, done.
Total 5 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.                                                                    
To github.com:silverscreen/lemons.git
   b52ae48..d36c124  master -> master
```

<br/>

### Step by Step

Actions can be observed under https://github.com/silverscreen/lemons/actions. From there I can see the Workflow file, as well as observe the job that is run.

A step is a set of tasks performed by a job, with each step in a job executing within the same virtual environment, allowing the actions in that job to share information using the filesystem.

Under build I can see that the following 4 steps took place:

```
1  Set up job                                           2s
2  Build appleboy/ssh-action@master                     5s
3  executing remote ssh commands using password         1s
4  Complete job                                         0s
```

#### 1. Build | Set up job

First, a virtual environment is conceived which in this case is `ubuntu-latest` as denoted by our workflow file. The GitHub Action `ssh-action` is then downloaded to that docker container.

```
Current runner version: '2.169.0'
Operating System
Virtual Environment
Prepare workflow directory
Prepare all required actions
Download action repository 'appleboy/ssh-action@master'
```

#### 2. Build | Build appleboy/ssh-action@master

The Action that was downloaded is then configured for use within our virtual environment.

```
Build container for action use: '/home/runner/work/_actions/appleboy/ssh-action/master/Dockerfile'.
/usr/bin/docker build -t 430c1a:9fc73e02d19840e39f81bbadf125e6d4 "/home/runner/work/_actions/appleboy/ssh-action/master"
Sending build context to Docker daemon  290.3kB

Step 1/4 : FROM appleboy/drone-ssh:1.5.6-linux-amd64
1.5.6-linux-amd64: Pulling from appleboy/drone-ssh
aad63a933944: Already exists
d2be78a94c0b: Pulling fs layer
f71add717aac: Pulling fs layer
45426c256262: Pulling fs layer
f71add717aac: Download complete
d2be78a94c0b: Download complete
45426c256262: Download complete
d2be78a94c0b: Pull complete
f71add717aac: Pull complete
45426c256262: Pull complete
Digest: sha256:98ee0ec37ef03a550f93b9b6a1a0075167a2efcf2d47d06ea03ce38850e182dc
Status: Downloaded newer image for appleboy/drone-ssh:1.5.6-linux-amd64
 ---> 3590d74bfbe7
Step 2/4 : ADD entrypoint.sh /entrypoint.sh
 ---> 8a57c2ee78a8
Step 3/4 : RUN chmod +x /entrypoint.sh
 ---> Running in 376a0d5717c6
Removing intermediate container 376a0d5717c6
 ---> a56e3b338ae3
Step 4/4 : ENTRYPOINT ["/entrypoint.sh"]
 ---> Running in e35***6bb53e8
```

#### 3. Build | executing remote ssh commands using password

Afterwards, our ssh commands are parsed by the action running in our docker environment, using the secret variables in our workflow (stored in **Settings>Secrets**), to establish an SSH connection with our host EC2 instance and execute `touch createdbygithubaction.txt`. 

```
/usr/bin/docker run --name c1a9fc73e02d19840e39f81bbadf125e6d4_1efb9d --label 430c1a --workdir /github/workspace --rm -e INPUT_HOST -e INPUT_USERNAME -e INPUT_KEY -e INPUT_PORT -e INPUT_SCRIPT -e INPUT_PASSPHRASE -e INPUT_PASSWORD -e INPUT_SYNC -e INPUT_TIMEOUT -e INPUT_COMMAND_TIMEOUT -e INPUT_KEY_PATH -e INPUT_PROXY_HOST -e INPUT_PROXY_PORT -e INPUT_PROXY_USERNAME -e INPUT_PROXY_PASSWORD -e INPUT_PROXY_PASSPHRASE -e INPUT_PROXY_TIMEOUT -e INPUT_PROXY_KEY -e INPUT_PROXY_KEY_PATH -e INPUT_SCRIPT_STOP -e INPUT_ENVS -e INPUT_DEBUG -e HOME -e GITHUB_JOB -e GITHUB_REF -e GITHUB_SHA -e GITHUB_REPOSITORY -e GITHUB_REPOSITORY_OWNER -e GITHUB_RUN_ID -e GITHUB_RUN_NUMBER -e GITHUB_ACTOR -e GITHUB_WORKFLOW -e GITHUB_HEAD_REF -e GITHUB_BASE_REF -e GITHUB_EVENT_NAME -e GITHUB_WORKSPACE -e GITHUB_ACTION -e GITHUB_EVENT_PATH -e RUNNER_OS -e RUNNER_TOOL_CACHE -e RUNNER_TEMP -e RUNNER_WORKSPACE -e ACTIONS_RUNTIME_URL -e ACTIONS_RUNTIME_TOKEN -e ACTIONS_CACHE_URL -e GITHUB_ACTIONS=true -e CI=true -v "/var/run/docker.sock":"/var/run/docker.sock" -v "/home/runner/work/_temp/_github_home":"/github/home" -v "/home/runner/work/_temp/_github_workflow":"/github/workflow" -v "/home/runner/work/lemons/lemons":"/github/workspace" 430c1a:9fc73e02d19840e39f81bbadf125e6d4
======CMD======
touch createdbygithubaction.txt
======END======
==============================================
✅ Successfully executed commands to all host.
==============================================
```

As stated in the build console (and observed by the green ticks), the commands are successfully executed against the host.

#### 4. Build | Complete job

Finally the job is completed.

```
Cleaning up orphan processes
```

</br>

Now when I SSH into my EC2 instance, I'll see that the file we created using our workflow now exists:

```
ubuntu@ubuntubox:~$ ls -al | grep createdbygit                                                                                           
-rw-rw-r--  1 ubuntu ubuntu     0 Apr 17 18:17 createdbygithubaction.txt  
```

<br/>

### About Self-hosted Runners

You can also use self-hosted runners with GitHub, just like you would with GitLab.

https://help.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners

Self-hosted runners offer more control of hardware, OS, and tools that GitHub-hosted runners provide, allowing you to choose;
* custom hardware configurations with more processing power or memory to run larger jobs
* install software available on the local network
* an OS not offerred by GitHub
* run local applications on that server

<br/>

### Adding Self-hosted Runners

To add a self-hosted runner to a user repository, you must be the repository owner.

https://help.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners

**Warning: We recommend that you do not use self-hosted runners with public repositories.**

For the purpose of this demonstration I'm going to build mine in AWS: 

```
lawrence@ulysses:~/git/Junos$ ssh ubuntu@35.177.XXX.XXX   
```

#### 1. Download Runner

On GitHub, navigate to the main page of the repository and go to **Settings>Actions** and click **Add runner**. 

Select the **operating system** of your self-hosted runner, in this case **Linux**. And then select the architecture of the runner machine, which is going to be **X64**.

#### Following the instructions:

```
// Create a folder
$ mkdir actions-runner && cd actions-runner
// Download the latest runner package
$ curl -O -L https://github.com/actions/runner/releases/download/v2.169.0/actions-runner-linux-x64-2.169.0.tar.gz
// Extract the installer
$ tar xzf ./actions-runner-linux-x64-2.169.0.tar.gz
```

Create a folder:

```
ubuntu@ubuntubox:~/actions-runner$ mkdir && cd actions-runner
```

Download the latest runner package: 

```                      
ubuntu@ubuntubox:~/actions-runner$ curl -O -L https://github.com/actions/runner/releases/download/v2.169.0/actions-runner-linux-x64-2.169.0.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   652  100   652    0     0  10516      0 --:--:-- --:--:-- --:--:-- 10516
100 72.0M  100 72.0M    0     0  10.3M      0  0:00:06  0:00:06 --:--:-- 11.8M
```

Extract the installer:

```
ubuntu@ubuntubox:~/actions-runner$ tar xzf ./actions-runner-linux-x64-2.169.0.tar.gz 
```

```
ubuntu@ubuntubox:~/actions-runner$ ls -al
...
drwxr-xr-x  3 ubuntu ubuntu    20480 Apr  8 17:02 bin
-rwxr-xr-x  1 ubuntu ubuntu     2671 Apr  8 17:00 config.sh
-rwxr-xr-x  1 ubuntu ubuntu      623 Apr  8 17:00 env.sh
drwxr-xr-x  4 ubuntu ubuntu     4096 Apr  8 17:01 externals
-rwxr-xr-x  1 ubuntu ubuntu     1666 Apr  8 17:00 run.sh
```

#### 2. Configure

```
// Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/silverscreen/lemons --token XXXXXXXXXXXXXXXXXXXXX
// Last step, run it!
$ ./run.sh
```

**Warning: This token will expire shortly after being generated onscreen. Failure to use it during that time will result in it in expiring and will require you to start the 'Add new runner' process again.**

Create the runner and start the configuration experience:

```
ubuntu@ubuntubox:~/actions-runner$ ./config.sh --url https://github.com/silverscreen/lemons --token XXXXXXXXXXXXXXXXXXXXX

--------------------------------------------------------------------------------
|        ____ _ _   _   _       _          _        _   _                      |
|       / ___(_) |_| | | |_   _| |__      / \   ___| |_(_) ___  _ __  ___      |
|      | |  _| | __| |_| | | | | '_ \    / _ \ / __| __| |/ _ \| '_ \/ __|     |
|      | |_| | | |_|  _  | |_| | |_) |  / ___ \ (__| |_| | (_) | | | \__ \     |
|       \____|_|\__|_| |_|\__,_|_.__/  /_/   \_\___|\__|_|\___/|_| |_|___/     |
|                                                                              |
|                       Self-hosted runner registration                        |
|                                                                              |
--------------------------------------------------------------------------------

# Authentication


√ Connected to GitHub

# Runner Registration

Enter the name of runner: [press Enter for ubuntubox] 

√ Runner successfully added
√ Runner connection is good

# Runner settings

Enter name of work folder: [press Enter for _work] 

√ Settings Saved.
```

Now finally, run it:

```
ubuntu@ubuntubox:~/actions-runner$ ./run.sh &
[1] 13946
ubuntu@ubuntubox:~/actions-runner$ 
√ Connected to GitHub

2020-04-21 15:20:27Z: Listening for Jobs
```

#### 3. Using your self-hosted runner

Use the following yaml in your workflow for each job to instruct the job to use the newly installed runner: 

```
runs-on: self-hosted
```

You should now see the newly added runner under **'Self-hosted runner'** within **Settings>Actions**, with a name, description, and status indicating that it is 'Idle'.

#### Which ports does a self hosted runner need?


The runners connect back to GitHub, no need for inbound firewall holes. Outbound HTTPS `TCP 443` is all that’s required.
