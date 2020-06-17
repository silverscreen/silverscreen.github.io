---
layout: post
title: Python and virtual environments
description: Python and virtual environments
summary: Python and virtual environments
comments: false
---

It is highly recommended that you use virtual environments for your Python projects. In this case, I had an issue with a VMware SDK that I was able to resolve after I setup a virtual environment.

```
$ virtualenv myenv
.. some output ..
$ source myenv/bin/activate
(myenv) $ pip install what-i-want
```

You should only use `sudo` or elevated permissions when you want to install stuff for the **global, system-wide Python installation**.

It is best to use a virtual environment which isolates packages for you. That way you can play around without polluting the global python install.

As a bonus, `virtualenv` does not need elevated permissions.

**It's best to not use the system-provided Python directly.** Leave that one alone since the OS can change it in undesired ways, as you experienced.

The best practice is to configure your own Python version(s) and manage them on a per-project basis using `virtualenv` (for Python 2) or `venv` (for Python 3). This eliminates all dependency on the system-provided Python version, and also isolates each project from other projects on the machine.

More information on `venv` here: https://docs.python.org/3/library/venv.html.

Each project can have a different Python point version if needed, and gets its own site_packages directory so pip-installed libraries can also have different versions by project. This approach is a major problem-avoider.

This was very informative (as always) https://realpython.com/python-virtual-environments-a-primer/

<br/>

### Using Virtual Environments

At its core, the main purpose of Python virtual environments is to create an isolated environment for Python projects. This means that each project can have its own dependencies, regardless of what dependencies every other project has.

The great thing about this is that there are no limits to the number of environments you can have since they’re just directories containing a few scripts. Plus, they’re easily created using the virtualenv or pyenv command line tools.

To get started, if you’re not using Python 3, you’ll want to install the virtualenv tool with pip:

```
$ pip install virtualenv
```

If you are using Python 3, then you should already have the `venv` module from the standard library installed.

Start by making a new directory to work with:

```
lawrence@cacti:~/ansible/vmware$
```

You can install venv with `python3-venv`:

```
lawrence@cacti:~/ansible/vmware$ sudo apt install python3-venv
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following packages were automatically installed and are no longer required:
  ieee-data libllvm7 python-certifi python-httplib2 python-jinja2 python-jmespath python-kerberos python-libcloud python-lockfile python-markupsafe
  python-netaddr python-paramiko python-pyasn1 python-requests python-selinux python-urllib3 python-xmltodict sshpass
Use 'sudo apt autoremove' to remove them.
The following additional packages will be installed:
  python3.6-venv
The following NEW packages will be installed
  python3-venv python3.6-venv
0 to upgrade, 2 to newly install, 0 to remove and 23 not to upgrade.
Need to get 7,396 B of archives.
After this operation, 44.0 kB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://gb.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 python3.6-venv amd64 3.6.9-1~18.04ubuntu1 [6,188 B]
Get:2 http://gb.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 python3-venv amd64 3.6.7-1~18.04 [1,208 B]
Fetched 7,396 B in 0s (63.5 kB/s)        
Selecting previously unselected package python3.6-venv.
(Reading database ... 240597 files and directories currently installed.)
Preparing to unpack .../python3.6-venv_3.6.9-1~18.04ubuntu1_amd64.deb ...
Unpacking python3.6-venv (3.6.9-1~18.04ubuntu1) ...
Selecting previously unselected package python3-venv.
Preparing to unpack .../python3-venv_3.6.7-1~18.04_amd64.deb ...
Unpacking python3-venv (3.6.7-1~18.04) ...
Setting up python3.6-venv (3.6.9-1~18.04ubuntu1) ...
Setting up python3-venv (3.6.7-1~18.04) ...
```

Now I should be able to create my `venv` environment:

```
lawrence@cacti:~/ansible/vmware$ python3 -m venv env
```

The Python 3 venv approach has the benefit of forcing you to choose a specific version of the Python 3 interpreter that should be used to create the virtual environment. This avoids any confusion as to which Python installation the new environment is based on.

From Python 3.3 to 3.4, the recommended way to create a virtual environment was to use the pyvenv command-line tool that also comes included with your Python 3 installation by default. But on 3.6 and above, python3 -m venv is the way to go.

Now we have a directory called `env` which contains the following:

```
lawrence@cacti:~/ansible/vmware$ ls -al env/
total 28
drwxrwxr-x 6 lawrence lawrence 4096 Jun  4 13:50 .
drwxrwxr-x 5 lawrence lawrence 4096 Jun  4 13:46 ..
drwxrwxr-x 2 lawrence lawrence 4096 Jun  4 13:50 bin
drwxrwxr-x 2 lawrence lawrence 4096 Jun  4 13:46 include
drwxrwxr-x 3 lawrence lawrence 4096 Jun  4 13:46 lib
lrwxrwxrwx 1 lawrence lawrence    3 Jun  4 13:46 lib64 -> lib
-rw-rw-r-- 1 lawrence lawrence   69 Jun  4 13:50 pyvenv.cfg
drwxrwxr-x 3 lawrence lawrence 4096 Jun  4 13:50 share
```

Here’s what each folder contains:

* bin: files that interact with the virtual environment
* include: C headers that compile the Python packages
* lib: a copy of the Python version along with a site-packages folder where each dependency is installed

More interesting are the **activate scripts** in the `bin` directory. These scripts are used to set up your shell to use the environment’s Python executable and its site-packages by default.

#### In order to use this environment’s packages/resources in isolation, you need to “activate” it. To do this, just run the following:

```
lawrence@cacti:~/ansible/vmware$ source env/bin/activate
(env) lawrence@cacti:~/ansible/vmware$ 
```

And you can see that now we are using python without our newly created evironment:

```
(env) lawrence@cacti:~/ansible/vmware$ which python
/home/lawrence/ansible/vmware/env/bin/python
```

```
(env) lawrence@cacti:~/ansible/vmware$ which python3
/home/lawrence/ansible/vmware/env/bin/python3
```

To `deactivate` you simply enter that exact command:

```
(env) lawrence@cacti:~/ansible/vmware$ deactivate 
lawrence@cacti:~/ansible/vmware$ 
```

```
lawrence@cacti:~/ansible/vmware$ which python
/usr/bin/python
lawrence@cacti:~/ansible/vmware$ which python3
/usr/bin/python3
```

You can even see that by calling the `$PATH` variable that our `*/env/bin` is now at the front of our path:

```
(env) lawrence@cacti:~/ansible/vmware$ echo $PATH
/home/lawrence/ansible/vmware/env/bin:/home/lawrence/.local/bin:/home/lawrence/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

Therefore it is now the first directory searched when running an executable on the command line, therefore our virtual environment's instance of python is used instead of our system-wide version.

<br/>

### Managing Virtual Environments With virtualenvwrapper

Whilst virtual environments solve some big problems with package management, they can begin to create some problems of their own once you have a few to manage. A tool `virtualenvwrapper` was created to solve this issues, and it's just some wrapper scripts around the main `virtualenv` tool, some features of which it provides are:

* It organises all of your virtual environments into one location.
* It provides methods to help easily create, delete, and copy environments.
* And provides a single command to switch between environments.

```
lawrence@cacti:~/ansible/vmware$ sudo pip3 install virtualenvwrapper
WARNING: The directory '/home/lawrence/.cache/pip' or its parent directory is not owned or is not writable by the current user. The cache has been disabled. Check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
Collecting virtualenvwrapper
  Downloading virtualenvwrapper-4.8.4.tar.gz (334 kB)
     |████████████████████████████████| 334 kB 1.3 MB/s 
Collecting virtualenv
  Downloading virtualenv-20.0.21-py2.py3-none-any.whl (4.7 MB)
     |████████████████████████████████| 4.7 MB 18.9 MB/s 
Collecting virtualenv-clone
  Downloading virtualenv_clone-0.5.4-py2.py3-none-any.whl (6.6 kB)
Collecting stevedore
  Downloading stevedore-2.0.0-py3-none-any.whl (42 kB)
     |████████████████████████████████| 42 kB 20.7 MB/s 
Collecting filelock<4,>=3.0.0
  Downloading filelock-3.0.12-py3-none-any.whl (7.6 kB)
Collecting distlib<1,>=0.3.0
  Downloading distlib-0.3.0.zip (571 kB)
     |████████████████████████████████| 571 kB 21.3 MB/s 
Requirement already satisfied: six<2,>=1.9.0 in /home/lawrence/.local/lib/python3.6/site-packages (from virtualenv->virtualenvwrapper) (1.15.0)
Collecting importlib-metadata<2,>=0.12; python_version < "3.8"
  Downloading importlib_metadata-1.6.0-py2.py3-none-any.whl (30 kB)
Collecting importlib-resources<2,>=1.0; python_version < "3.7"
  Downloading importlib_resources-1.5.0-py2.py3-none-any.whl (21 kB)
Collecting appdirs<2,>=1.4.3
  Downloading appdirs-1.4.4-py2.py3-none-any.whl (9.6 kB)
Requirement already satisfied: pbr!=2.1.0,>=2.0.0 in /usr/lib/python3/dist-packages (from stevedore->virtualenvwrapper) (3.1.1)
Collecting zipp>=0.5
  Downloading zipp-3.1.0-py3-none-any.whl (4.9 kB)
Building wheels for collected packages: virtualenvwrapper, distlib
  Building wheel for virtualenvwrapper (setup.py) ... done
  Created wheel for virtualenvwrapper: filename=virtualenvwrapper-4.8.4-py2.py3-none-any.whl size=25542 sha256=14d56e0ae8fb813316376f0f52a83d136efe13e41357dd4cb378a4754af3796e
  Stored in directory: /tmp/pip-ephem-wheel-cache-7fa2l1px/wheels/e8/a1/ce/5571936b486922776c19d300086f93c263c0f039c555ad537a
  Building wheel for distlib (setup.py) ... done
  Created wheel for distlib: filename=distlib-0.3.0-py3-none-any.whl size=336972 sha256=f7bc03edeb6cea6df10339763c8c001489dd7daae2a14edc99c635fddbbcb0df
  Stored in directory: /tmp/pip-ephem-wheel-cache-7fa2l1px/wheels/33/d9/71/e4e3cac73529e1947df418af0f140cd7589d5d9ec0e17ecfc2
Successfully built virtualenvwrapper distlib
Installing collected packages: filelock, distlib, zipp, importlib-metadata, importlib-resources, appdirs, virtualenv, virtualenv-clone, stevedore, virtualenvwrapper
Successfully installed appdirs-1.4.4 distlib-0.3.0 filelock-3.0.12 importlib-metadata-1.6.0 importlib-resources-1.5.0 stevedore-2.0.0 virtualenv-20.0.21 virtualenv-clone-0.5.4 virtualenvwrapper-4.8.4 zipp-3.1.0
```

Now it is installed we'll need to activate its shell functions, this can be done by running `source` on the installed `virtualenvwrapper.sh` script, the location of which can be found with `which` (pun intended):

```
lawrence@cacti:~/ansible/vmware$ which virtualenvwrapper.sh 
/usr/local/bin/virtualenvwrapper.sh
```

Now we can add the following three lines to our shell's startup file, in my case I'm using Bash therefore I would add these within `~/.bashrc`:

```
export WORKON_HOME=$HOME/.virtualenvs   # Optional
export PROJECT_HOME=$HOME/projects      # Optional
source /usr/local/bin/virtualenvwrapper.sh
```

Now reload the startup file:

```
source ~/.bashrc
```

If you run into this error:

```
lawrence@cacti:~$ source ~/.bashrc 
/usr/bin/python: No module named virtualenvwrapper
virtualenvwrapper.sh: There was a problem running the initialization hooks.

If Python could not import the module virtualenvwrapper.hook_loader,
check that virtualenvwrapper has been installed for
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python and that PATH is
set properly.
```

Then we need to change `export VIRTUALENVWRAPPER_PYTHON=/usr/local/bin/python` to:

```
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
```

So I have amended my `~/.bashrc` file to:

```
export WORKON_HOME=$HOME/.virtualenvs   # Optional
export PROJECT_HOME=$HOME/projects      # Optional
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source /usr/local/bin/virtualenvwrapper.sh
```

And now it should it work:

```
lawrence@cacti:~$ source ~/.bashrc 
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/premkproject
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/postmkproject
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/initialize
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/premkvirtualenv
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/postmkvirtualenv
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/prermvirtualenv
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/postrmvirtualenv
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/predeactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/postdeactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/preactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/postactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/get_env_details
```

Credit goes to https://medium.com/@gitudaniel/installing-virtualenvwrapper-for-python3-ad3dfea7c717 for helping me through this.

Now I should have a directory located at `$WORKON_HOME` that contains all of the virtualenvwrapper files:

```
lawrence@cacti:~$ echo $WORKON_HOME
/home/lawrence/.virtualenvs
```

We should now also have shell commands available to help us manage the environments, such as:

* `workon`
* `deactivate`
* `mkvirtualenv`
* `cdvirtualenv`
* `rmvirtualenv`

For more info on commands, installation, and configuring virtualenvwrapper, check out the official documentation.
* https://virtualenvwrapper.readthedocs.io/en/latest/install.html

<br/>

### Starting a new project

Anytime I want to start a new project now all I need to do is use `mkvirtualenv <project-name>`:

```
lawrence@cacti:~$ mkvirtualenv vmware-project
created virtual environment CPython3.6.9.final.0-64 in 193ms
  creator CPython3Posix(dest=/home/lawrence/.virtualenvs/vmware-project, clear=False, global=False)
  seeder FromAppData(download=False, pip=latest, setuptools=latest, wheel=latest, via=copy, app_data_dir=/home/lawrence/.local/share/virtualenv/seed-app-data/v1.0.1)
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/vmware-project/bin/predeactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/vmware-project/bin/postdeactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/vmware-project/bin/preactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/vmware-project/bin/postactivate
virtualenvwrapper.user_scripts creating /home/lawrence/.virtualenvs/vmware-project/bin/get_env_details
(vmware-project) 
```

This creates and **activates** a new environment in the directory located with `$WORKON_HOME` where all virtualenvwrapper envs will are stored:

```
(vmware-project) lawrence@cacti:~$
```

```
(vmware-project) lawrence@cacti:~$ ls -al .virtualenvs/
total 60
drwxrwxr-x  3 lawrence lawrence 4096 Jun  4 14:36 .
drwxr-xr-x 42 lawrence lawrence 4096 Jun  4 14:26 ..
-rwxr-xr-x  1 lawrence lawrence  135 Jun  4 14:26 get_env_details
-rw-r--r--  1 lawrence lawrence   96 Jun  4 14:26 initialize
-rw-r--r--  1 lawrence lawrence   73 Jun  4 14:26 postactivate
-rw-r--r--  1 lawrence lawrence   75 Jun  4 14:26 postdeactivate
-rwxr-xr-x  1 lawrence lawrence   66 Jun  4 14:26 postmkproject
-rw-r--r--  1 lawrence lawrence   73 Jun  4 14:26 postmkvirtualenv
-rwxr-xr-x  1 lawrence lawrence  110 Jun  4 14:26 postrmvirtualenv
-rwxr-xr-x  1 lawrence lawrence   99 Jun  4 14:26 preactivate
-rw-r--r--  1 lawrence lawrence   76 Jun  4 14:26 predeactivate
-rwxr-xr-x  1 lawrence lawrence   91 Jun  4 14:26 premkproject
-rwxr-xr-x  1 lawrence lawrence  220 Jun  4 14:26 premkvirtualenv
-rwxr-xr-x  1 lawrence lawrence  111 Jun  4 14:26 prermvirtualenv
drwxrwxr-x  4 lawrence lawrence 4096 Jun  4 14:36 vmware-project
```

To **stop** using that environment, you just need to `deactivate` it like before:

```
(vmware-project) lawrence@cacti:~$ deactivate 
lawrence@cacti:~$ 
```

If you have several environments to choose from, you can **list them** all with the `workon` command:

```
lawrence@cacti:~$ workon
vmware-project
```

And finally to activate a project simply give the name of the project to `workon`:

```
lawrence@cacti:~$ workon vmware-project
(vmware-project) lawrence@cacti:~$
```

<br/>

### Switching between python versions

You can switch between python versions in `virtualenv` using the `-p` parameter, combined with `$(which )`you can select which version of python to use. For example, to use python3:

```
lawrence@cacti:~$ virtualenv -p $(which python3) vmware-project
created virtual environment CPython3.6.9.final.0-64 in 217ms
  creator CPython3Posix(dest=/home/lawrence/vmware-project, clear=False, global=False)
  seeder FromAppData(download=False, pip=latest, setuptools=latest, wheel=latest, via=copy, app_data_dir=/home/lawrence/.local/share/virtualenv/seed-app-data/v1.0.1)
  activators BashActivator,CShellActivator,FishActivator,PowerShellActivator,PythonActivator,XonshActivator
(vmware-project) 
```

<br/>

Hopefully this has introduced you to the concept of virtual environments within Python. There are several different ways to achieve this such as `pyvenv`, `virtualenv`, anaconda (`conda`) - so go check them out and find one that suits how you work!