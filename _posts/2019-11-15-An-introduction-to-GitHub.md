---
layout: post
title: An introduction to GitHub
description: An introduction to GitHub
summary: An introduction to GitHub
comments: false
---

Recently I've been running a few workshops on Git and its mechanics. I am by no means a 'Pro' but I would consider myself well enough acquainted that if you are a complete 'noob' then you should find this helpful.

Git is a version control system, once an actively maintained open source project originally developed in 2005 by Linus Torvald. If you're familiar with Linux, then that name should ring a bell.There are two variations of 'git', one being **GitHub**  and **GitLab**.

**GitHub** is a git repository and collaboration platform to help developers communicate and manage their code. It has features such as an **issue tracker**, **bug tracking system**, **source code management**, and its own **built-in CI/CD tool** known as **Actions**. It also supports **wikis** as well as **markdown-based** documentation.

**GitLab** comes with pretty much all the features Github offers, however, GitLab was initially designed with a built-in **CI/CD tool**. This CI/CD integration makes GitLab the preferred choice for organisations, along with the fact that it can be hosted internally.

<br/>

----

### 1.0 Building Your First Repo 

A repo is ...

So first we're going to start out by creating a new public repo. Repos can be either Public or Private but not both. 

For the purpose of this demonstration I'm going to call this repo `git-example`.

1. **Create a new repository**
2. Repository name: `git-example`
3. Description: An example repo for demonstrating the use of git
4. Public
5. Initialize this repository with a README, this will let me immediately clone the repository to my computer
6. **Create repository**

The `.gitignore` file is a text file that tells Git which files or folders to ignore in a project. 

For example: 

```
# Ignore all text files
*.txt

# Ignore files related to API keys
.env
```

This would be particularly useful if you didn't want to risk uploading a private API key stored in a file. 

This file can be editted or change your **global** `.gitignore file` which will omit anything you specify for all repos. 

Any ammendments to this file will require clearing the cache: 

```
git rm -r --cached
```

**Note that this will also lose any tracked file changes so be sure to commit those changes before you do so.**

<br/>

### 2.0 Cloning Your Repo

I can clone a repo with either HTTPS or SSH. The former using requiring my GitHub Email and Password (if the repo was private) and the latter needing my SSH key to be added to my GitHub profile beforehand. 

So I'll clone my repo to my local machine: 

```
lawrence@cacti:~/workshops$ git clone git@github.com:silverscreen/git-example.git
Cloning into 'git-example'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (3/3), done.
```

And now I have a local copy of this public repo stored on my machine, where we have our `README.md` file that we chose to initialize our repo with:

```
lawrence@cacti:~/workshops/git-example$ ls
README.md
```

There's also a hidden folder named `.git`:

```
lawrence@cacti:~/workshops$ tree -a git-example/
git-example/
├── .git
│   ├── branches
│   ├── config
│   ├── description
│   ├── HEAD
│   ├── hooks
│   │   ├── applypatch-msg.sample
│   │   ├── commit-msg.sample
│   │   ├── fsmonitor-watchman.sample
│   │   ├── post-update.sample
│   │   ├── pre-applypatch.sample
│   │   ├── pre-commit.sample
│   │   ├── prepare-commit-msg.sample
│   │   ├── pre-push.sample
│   │   ├── pre-rebase.sample
│   │   ├── pre-receive.sample
│   │   └── update.sample
│   ├── index
│   ├── info
│   │   └── exclude
│   ├── logs
│   │   ├── HEAD
│   │   └── refs
│   │       ├── heads
│   │       │   └── master
│   │       └── remotes
│   │           └── origin
│   │               └── HEAD
│   ├── objects
│   │   ├── info
│   │   └── pack
│   │       ├── pack-c562f6caabc2edca52c969707638e2d30f0c8787.idx
│   │       └── pack-c562f6caabc2edca52c969707638e2d30f0c8787.pack
│   ├── packed-refs
│   └── refs
│       ├── heads
│       │   └── master
│       ├── remotes
│       │   └── origin
│       │       └── HEAD
│       └── tags
└── README.md

17 directories, 25 files
```

You'll notice that there are alot of files stored within `.git` which is hidden by default. This enables git to work with your repo, so if for whatever reason you wanted to add the contents of this repo to another repo, you would simply delete this folder. 

<br/>

### 3.0 Branch And Remote Terminology

So now I have my repo installed, I can begin working with it locally. I can use `git status` to show what's happening with `git add` and `git commit`:

```
lawrence@cacti:~/workshops$ cd git-example/
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

nothing to commit, working tree clean
```

The `git status` command displays the **state** of the my working directory and the staging area. It lets you see which changes have been staged with `git add`, and which haven't.

I can see from the `git status` output that: 
* I'm working within the `master` branch.
* My branch is up-to-date with `'origin/master'`.
* And that there is `nothing to commit`.

Now it's really important that you understand the terminology used here:

`master` refers to our **local branch**.

`origin/master` is referring to the **remote branch**, which is a local copy of the branch named `'master'` on the remote named `'origin'`. 

`origin` being just an **alias** on your machine for a particular remote repository.

**Remotes** are simply an alias that store the URL of repositories. You can see what URL belongs to each remote by doing: 

```
lawrence@cacti:~/workshops/git-example$ git remote -v
origin  git@github.com:silverscreen/git-example.git (fetch)
origin  git@github.com:silverscreen/git-example.git (push)
```

For example, **remotes** and URLs can be used interchangeably like so:

```
lawrence@cacti:~/workshops/git-example$ git push origin master 
```

Is the same as:

```
lawrence@cacti:~/workshops/git-example$ git push git@github.com:silverscreen/git-example.git master
```

<br/>

### 4.0 Staging

One of the key differences between git and other version control systems, is that commits can be staged locally without having to be committed to the master remote repository. With the staging area acting as a buffer between the working directory and the project history.

First let's add some code to our repo:

```
lawrence@cacti:~/workshops/git-example$ mkdir bash && echo 'echo $HOME' >> bash/printHOME.sh
lawrence@cacti:~/workshops/git-example$ chmod +x bash/printHOME.sh 
lawrence@cacti:~/workshops/git-example$ ./bash/printHOME.sh 
/home/lawrence
```

Now that I have made amendments to my local repo, I should be able to see these changes by calling `git status`: 

```
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        bash/

nothing added to commit but untracked files present (use "git add" to track)
```

From the above output, it states that the following files are `Untracked`. This means that git sees a file you didn't have from a previous commit and will not include it in your commit snapshots until you add it using `git add`: 

```
lawrence@cacti:~/workshops/git-example$ git add .
lawrence@cacti:~/workshops/git-example$ 
```

Notice that I append `git add` with `.` to indicate that I want to **add everything** in the current working directory (this includes any and all child directories).

Now when I call `git status` again, git recognises that I want to add my file to my next commit. What I am effectively doing here is **staging** my files to be committed to my remote repo.

To remove a specific file (or directory) from staging, I can use following command:

```
lawrence@cacti:~/workshops/git-example$ git rm --cached bash -r
rm 'bash/printHOME.sh'
```

And now running `git status` again confirms that my directory is nolonger staged: 

```
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        bash/

nothing added to commit but untracked files present (use "git add" to track)
```

<br/>

### 5.0 Commits

To capture a repo's staged changes, `git commit` creates a snapshot of the project prior to pushing them.

Commits can be thought of as snapshots or milestones along the timeline of a project, used to capture the state of a project at that point in time. With these snapshots always being committed to the local repository. 

It also lets developers work in an isolated environment, deferring integration until they’re at a convenient point to merge with other users. While isolation and deferred integration are individually beneficial, it is in a team's best interest to integrate frequently and in small units.

You can `commit` your staged changes to your local repository with: 

```
lawrence@cacti:~/workshops/git-example$ git add .
lawrence@cacti:~/workshops/git-example$ git commit -m "Added bash"
[master 0a2ad54] Added bash
 1 file changed, 1 insertion(+)
 create mode 100755 bash/printHOME.sh
```

The command `git commit` saves my changes to the local repository and the parameter `-m` sets the commit's message `"Added bash"`. 

Make sure to always provide a **concise description** that helps your teammates (and yourself) understand what happened during your commit.

You can clearly see now that are local commits have been saved and are ahead of our remote branch `origin/master` by 1 commit:

```
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

### 5.1 To Undo A Commit

You can undo the last `git commit` with `reset`.

This should be used with the `--soft` parameter to preserve changes done to your files.

```
lawrence@cacti:~/workshops/git-example$ git reset --soft HEAD~1
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   bash/printHOME.sh
```

You'll notice that I also had to specify the commit to undo which is `HEAD~1` in this case as denoted here with `git log --oneline`:

```
lawrence@cacti:~/workshops/git-example$ git log --oneline
341b0d8 (HEAD -> master) Added bash
021ef73 (origin/master, origin/HEAD) Initial commit
```

Once I have performed a soft `reset`, you'll see that my files are still tracked from when we added them and have yet to be committed: 

```
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   bash/printHOME.sh
```

<br/>

### 6.0 Pushing

The `git push` command is used to upload **local** content previously committed with the `git commit` command, to the **remote repository**. 

Pushing is how you transfer commits, and is the counterpart to `git fetch`, where `fetch` **imports** commits to **local** branches, `push` **exports** commits to **remote** branches.

To proceed, ensure you have your changes committed:

```
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
```

Our `branch is ahead of 'origin/master'` which means we have changes that differ from the remote master repo. 

So now we can push our changes to `origin/master` with `git push`:

```
lawrence@cacti:~/workshops/git-example$ git push
Counting objects: 4, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 345 bytes | 345.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To github.com:silverscreen/git-example.git
   021ef73..54c99c4  master -> master
```

### 7.0 Working With Commits

So now my **remote branch** reflects that of my **local branch**, and those changes can now be viewed on GitHub and can be identified as the following commit ID `54c99c4b4dc222df35ec8078ee0bd4d5638594a3`. 

A git commit ID is a SHA-1 hash of every important thing about the commit. This includes: 

* The content, all of it, not just the diff.
* Commit date.
* Committer's name and email address.
* Log message.
* The ID of the previous commit(s).

By default, with no arguments, `git log` lists the commits made in that repository in reverse chronological order, with the most recent commit at the top. This lists each commit with its **SHA-1 checksum**, the author’s **name** and **email**, the **date** written, and the **commit message**:

```
lawrence@cacti:~/workshops/git-example$ git log
commit 54c99c4b4dc222df35ec8078ee0bd4d5638594a3 (HEAD -> master, origin/master, origin/HEAD)
Author: Lawrence Long <4d7vs8@gmail.com>
Date:   Tue May 5 13:23:27 2020 +0100

    Added bash script example

commit 021ef73274abe2c80b45795fd9adc20b8d7917a8
Author: Lawrence <34809422+silverscreen@users.noreply.github.com>
Date:   Mon May 4 13:46:47 2020 +0100

    Initial commit
```    

**Commit IDs can be abbreviated** as long as that partial hash is at least four characters long and unambiguous. These abbreviated versions can be shown with the `--abbrev-commit` option:

```
lawrence@cacti:~/workshops/git-example$ git log --abbrev-commit
commit 54c99c4 (HEAD -> master, origin/master, origin/HEAD)
Author: Lawrence Long <4d7vs8@gmail.com>
Date:   Tue May 5 13:23:27 2020 +0100

    Added bash script example

commit 021ef73
Author: Lawrence <34809422+silverscreen@users.noreply.github.com>
Date:   Mon May 4 13:46:47 2020 +0100

    Initial commit
```

You can then refer to those commits simply by passing the short version to `git show`: 

```
lawrence@cacti:~/workshops/git-example$ git show 54c99
commit 54c99c4b4dc222df35ec8078ee0bd4d5638594a3 (HEAD -> master, origin/master, origin/HEAD)
Author: Lawrence Long <4d7vs8@gmail.com>
Date:   Tue May 5 13:23:27 2020 +0100

    Added bash script example

diff --git a/bash/printHOME.sh b/bash/printHOME.sh
new file mode 100755
index 0000000..3fb149f
--- /dev/null
+++ b/bash/printHOME.sh
@@ -0,0 +1 @@
+echo $HOME
```

<br/>

### 7.1 Revert A Commit

Maybe you want to revert a commit


To revert to a previous commit, use `git revert`, specifying the commit you wish to revert:

```
lawrence@cacti:~/workshops/git-example$ git revert 54c99c4
```

You'll then need to confirm the following:

```

Revert "Added bash script example"

This reverts commit 54c99c4b4dc222df35ec8078ee0bd4d5638594a3.

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Your branch is up-to-date with 'origin/master'.
#
# Changes to be committed:
#       deleted:    bash/printHOME.sh
#
```

Save and exit. Now you are ready to push your changes: 

```
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
lawrence@cacti:~/workshops/git-example$ git push
Counting objects: 2, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (1/1), done.
Writing objects: 100% (2/2), 272 bytes | 272.00 KiB/s, done.
Total 2 (delta 0), reused 1 (delta 0)
To github.com:silverscreen/git-example.git
   54c99c4..291ba85  master -> master
```

And now `origin/master` should be back to the same state it was prior to commit `54c99c4`:

```
lawrence@cacti:~/workshops/git-example$ ls
README.md
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

nothing to commit, working tree clean
```

<br/>

### 7.0 Branching

Git also features branching. Branches are effectively a unique set of code changes with a name to identify them from the master branch. The master branch being the one where all changes get merged together. So if you were to add a new feature or fix a bug, you would typically spawn a new branch to encompass these changes.

This can help prevent unstable code from getting merged into the main code base.

**Do not mess with the master.** If you make changes to the master branch of a group project while other people are also working on it, your on-the-fly changes will ripple out to affect everyone else and very quickly there will be merge conflicts, weeping, rending of garments, and plagues of locusts.

#### Testing Branches

If you want to test a previous commit just do `git checkout` followed by the commit ID. This puts you in a 'detached HEAD' state where you can test the last working version of your project:

```
lawrence@cacti:~/workshops/git-example$ git checkout 54c99c4
Note: checking out '54c99c4'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 54c99c4 Added bash script example
lawrence@cacti:~/workshops/git-example$ git status
HEAD detached at 54c99c4
nothing to commit, working tree clean
lawrence@cacti:~/workshops/git-example$ ls
bash  README.md
```

This makes my working directory match the exact state of the `54c99c4` commit. 

Now I can look at files, compile the project, run tests, and even edit files without worrying about losing the current state of the project. **Nothing you do in here will be saved in your repository.** 

To return to developing on a particular branch, you need to specify the branch you're working on:

```
lawrence@cacti:~/workshops/git-example$ git checkout master
Previous HEAD position was 54c99c4 Added bash script example
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
```

```
lawrence@cacti:~/workshops/git-example$ git status
On branch master
Your branch is up-to-date with 'origin/master'.

nothing to commit, working tree clean
lawrence@cacti:~/workshops/git-example$ ls
README.md
```

<br/>

### Creating A Test Branch

Lets say we want to revert our most recent commit and create a test branch from it, use `git checkout -b <BRANCH-NAME>`:

```
lawrence@cacti:~/workshops/git-example$ git checkout -b test-branch
Switched to a new branch 'test-branch'
```

Now I'll revert my changes back to when my bash code was present: 

```
lawrence@cacti:~/workshops/git-example$ git revert 37cf
[test-branch ccd5d2f] Revert "Revert "Revert "Revert "Added bash script example""""
 1 file changed, 1 insertion(+)
 create mode 100755 bash/printHOME.sh
```

```
lawrence@cacti:~/workshops/git-example$ ls
bash  README.md
lawrence@cacti:~/workshops/git-example$ git status
On branch test-branch
nothing to commit, working tree clean
```

So now I'm on a working branch named `test-branch` with the files code from a previous commit.

To **view branches** in a Git repository, run the command `git branch`:

```
lawrence@cacti:~/workshops/git-example$ git branch
* master
  test-branch
```  

Now lets work on our `test-branch`:

```
lawrence@cacti:~/workshops/git-example$ mkdir python
lawrence@cacti:~/workshops/git-example$ echo "print('some python')" >> python/python.py
```

Now lets stage our changes: 

```
lawrence@cacti:~/workshops/git-example$ git add .
lawrence@cacti:~/workshops/git-example$ git status
On branch test-branch
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   python/python.py
```

```
lawrence@cacti:~/workshops/git-example$ git log --pretty=oneline
ccd5d2f84b1006ac6f890e0d192a2c2b1e60be18 (HEAD -> test-branch) Revert "Revert "Revert "Revert "Added bash script example""""
37cf2bfba1d55e7fe54dac13b7635cc8c42306fc (origin/master, origin/HEAD, master) Revert "Revert "Revert "Added bash script example"""
f28e038f85449ac522c496c885f8eae1a8644586 Revert "Revert "Added bash script example""
291ba85b4f0684d901ba847d5f6fa99e3e463ee0 Revert "Added bash script example"
54c99c4b4dc222df35ec8078ee0bd4d5638594a3 Added bash script example
021ef73274abe2c80b45795fd9adc20b8d7917a8 Initial commit
```

```
lawrence@cacti:~/workshops/git-example$ git commit -m "revert to 37cf2b + python"
[test-branch 81ee72c] revert to 37cf2b + python
 1 file changed, 1 insertion(+)
 create mode 100644 python/python.py
```

Now that I've commited my changes to `test-branch`, I need to push my new branch to my remote repo as right now it only exists **locally**:

```
lawrence@cacti:~/workshops/git-example$ git push
fatal: The current branch test-branch has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin test-branch
```

So normally in a standard setup, you generally have an `origin` and an `upstream` **remote**, the `upstream` being the gatekeeper of the project or the source of truth to which you wish to contribute.

#### What is an upstream?

An upstream is simply another branch name, usually a remote-tracking branch, associated with a (regular, local) branch.

Every branch has the option of having one (1) upstream set. That is, every branch either has an upstream, or does not have an upstream. No branch can have more than one upstream.

Someone on stackoverflow wrote a really good post that goes quite indepth on this particular topic https://stackoverflow.com/questions/37770467/why-do-i-have-to-git-push-set-upstream-origin-branch. 

#### How come master already has an upstream set?

When you first clone from some remote, using `git clone`, now you have a remote-tracking branch named `origin/master`, because you just cloned it.

Git assumes that you must have meant: "make me a new local master that points to the same commit as remote-tracking origin/master, and, while you're at it, set the upstream for master to origin/master." 

This happens for every branch you git checkout that you do not already have. Git creates the branch and makes it "track" (have as an upstream) the corresponding remote-tracking branch.

But this doesn't work for new branches, i.e., branches with no remote-tracking branch - yet.

There is, as yet, no `origin/test-branch` therefore your local repo cannot track remote-tracking branch `origin/test-branch` because **it does not currently exist**.

To set it now, rather than during the first push, use: 

```
lawrence@cacti:~/workshops/git-example$ git branch --set-upstream-to origin/test-branch
Branch 'test-branch' set up to track remote branch 'test-branch' from 'origin'.
```

Or the much shorter: 

```
lawrence@cacti:~/workshops/git-example$ git branch -u origin/test-branch
Branch 'test-branch' set up to track remote branch 'test-branch' from 'origin'.
```

Now with that out the way, we can finally push to our remote branch:

```
lawrence@cacti:~/workshops/git-example$ git push --set-upstream origin test-branch
Counting objects: 8, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (8/8), 665 bytes | 665.00 KiB/s, done.
Total 8 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
remote: 
remote: Create a pull request for 'test-branch' on GitHub by visiting:
remote:      https://github.com/silverscreen/git-example/pull/new/test-branch
remote: 
To github.com:silverscreen/git-example.git
 * [new branch]      test-branch -> test-branch
Branch 'test-branch' set up to track remote branch 'test-branch' from 'origin'.
```

<br/>

### 8.0 Merge Requests

In the output, git will prompt you with a direct link for creating a **merge request**:

```
remote: Create a pull request for 'test-branch' on GitHub by visiting:
remote:      https://github.com/silverscreen/git-example/pull/new/test-branch
```

Now copy that link and paste it in your browser, and the **New Merge Request** page will be displayed.

Here we can begin filling in our merge request which will put in a request to merge `test-branch` with our `master` branch. 

Be concise with a clear description of your changes and then submit your request by hitting **Create pull request**. 


Git will automatically begin checking if the two branches can be merged, from which we should receive no conflicts with the base branch and thus be able to hit **Merge pull request**.

```
Pull request successfully merged and closed
You’re all set—the test-branch branch can be safely deleted.
```

And now our `master` branch reflects the changes we made in `test-branch`. 

<br/>

### 9.0 Pull

So now if we go back to our `master` branch: 

```
lawrence@cacti:~/workshops/git-example$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
lawrence@cacti:~/workshops/git-example$ ls
README.md
```

We can now use `git pull` to pull our changes from the **remote** to our local repo: 

```
lawrence@cacti:~/workshops/git-example$ git pull
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (1/1), done.
From github.com:silverscreen/git-example
   37cf2bf..9e95166  master     -> origin/master
Updating 37cf2bf..9e95166
Fast-forward
 bash/printHOME.sh | 1 +
 python/python.py  | 1 +
 2 files changed, 2 insertions(+)
 create mode 100755 bash/printHOME.sh
 create mode 100644 python/python.py
``` 

<br/>

### 10.0 Branch Settings

Let's assume we're working with someone else on this project, and we've agreed that any changes should be reviewed and approved before they're committed to the main branch.  

We're going to add a rule that requires a pull request to be reviewed before merging and apply this to the `master` branch. 

https://help.github.com/en/github/administering-a-repository/configuring-protected-branches

Within **Settings>Branches** you have the choice of configuring one or more **Branch protection rules** Next to "Branch protection rules", click **Add rule** "Branch name pattern", type the branch name or pattern you want to protect which in this case is going to be `master`. 

Then we're going to enable **Require pull request reviews before merging**. 

With this enabled, all commits must be made to a **non-protected branch** such as `test-branch` and submitted via a **pull request** with the required number of approving reviews and no changes requested before it can be merged with `master`. 

Lastly, save your changes, and now we can begin adding someone new to our project to test this in action. 

<br/>

Settings>Manage access

Then underneath **Who has access** click **Invite a collaborator** and begin searching for the relevant user. Once you've added them they should have **Pending Invite** next to their username which they'll need to accept the invitation `https://github.com/silverscreen/git-example/invitations` before they can gain access to the project. 

Once they've accepted they should now have the status **Collaborator** next to their username which means they're ready to start collaborating with you on your project. 

Our remote user has made the following changes to one of our files:

```
remoteuser@cacti:~/git-example$ cat bash/printHOME.sh 
#!/bin/bash

# Add two numeric value
((sum=25+35))

#Print the result
echo $sum
```

They've now committed and pushed their changes to `test-branch`: 

```
remoteuser@cacti:~/git-example$ git push
Counting objects: 4, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 441 bytes | 441.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To github.com:silverscreen/git-example.git
   81ee72c..6f7b13b  test-branch -> test-branch
```

And now they can make open a pull request to merge `test-branch` with master like they did earlier.

This time however, the user submitting the pull request will not be able to approve it: 

```
Review required
At least 1 approving review is required by reviewers with write access. 
Merging is blocked
Merging can be performed automatically with 1 approving review.
```

This protection was implemented through our rule, so now it is up to us to review and approve/deny the changes before it can be merged with `master`.

So now we'll navigate to `https://github.com/silverscreen/git-example/pulls` and examine would should be our second merge request so far `Ammended printHome.sh #2`. 

Here I can **review** the changes, **write a comment** "These look good! Approved =)", tick **Approve** and then **Submit review**.

Now either myself or the collaborator can hit **Merge pull request** and merge the new changes from `test-branch` into `master`.

It's that easy!

