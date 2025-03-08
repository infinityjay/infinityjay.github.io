---
layout: post
title:  Git - learning notes
categories:
  - CI/CD
tags:
  - Git
  - Version control
  - Learning notes
---
Content

{% include toc %}

# Introduction

git: Distributed version control system

## Install git

> Linux system

```bash
# First check if git is installed
git
The program 'git' is currently not installed. You can install it by typing:
# Complete the installation according to the above command, --Debian or Ubuntu Linux use the following command to install
sudo apt-get install git
```

> Mac OS system

```bash
# Install via homebrew
brew install git

# Or download Xcode, you can install git in the toolbox in Xcode
```

## Create a version library

* step1 First create an empty directory

```bash
# Create an empty directory
$ mkdir learngit
$ cd learngit
$ pwd
/Users/michael/learngit
```

* step 2 Initialize the directory as a Git repository

```bash
$ git init
# After successful initialization, the output is as follows
Initialized empty Git repository in /Users/michael/learngit/.git/
```

After the initialization is successful, there will be an additional .git directory in the directory

> Add files to the repository

Create a new file readme.txt in the `learngit` directory (or its subdirectory)

* step1 Add files to the repository

```bash
git add readme.txt
```

* step2 Submit files and add submission instructions with -m

```bash
git commit -m "wrote a readme file"
```

Note: You can `add` different files multiple times, and `commit` can submit multiple files at a time

```bash
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."
```

# Usage tutorial

## Version rollback

> View the history of the version control system

```bash
git log
```

This command displays the commit log from the most recent to the most distant.

The `commit id` in Git is not 1, 2, 3, but a very large number represented in hexadecimal. If everyone's updated version id is represented by 1, 2, 3, it will cause version id conflicts.

> Representation of each version

`HEAD`: represents the current version;

`HEAD^`: represents the previous version;

`HEAD^^`: represents the previous version;

`HEAD~100`: represents the previous 100 versions;

```bash
# Roll back to the previous version
git reset --hard HEAD^

# Go back to a future version (using the version number, only the first few digits of the version number are needed)
git reset --hard <version number (the first few digits are enough)>
```

Git's version rollback speed is very fast, because Git has an internal `HEAD` pointer pointing to the current version. When you roll back the version, Git just changes HEAD from pointing to the current version to pointing to the specified version.

`git reflog`: Record every command. After closing the git window, you can view the historical commands to find the id and other information of the rollback version;

`git status`: View the status of the temporary storage area (stage), that is, the area before commit after executing the `git add` command

![image-20210723151023753](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210723151023753.png)

## Remote repository

### GitHub

> Configure SSH key

The transmission between Github and the local repository is encrypted via SSH.

* Create SSH key

In the user's home directory, check whether there is a `.ssh` directory, which contains two files, `id_rsa` and `id_rsa.pub`. id_rsa stores the private key and cannot be leaked. id_rsa.pub is the public key and can be told to anyone.

* Set up SSH keys in Github

In the account settings of GitHub, open the "SSH Keys" page, click "Add SSH key", and paste the content of `id_rsa.pub` in the text box

> Connect to the remote repository

First, create a new repository in GitHub, and then execute the following command under the path of the local repository

```bash
git remote add origin git@github.com:xxx/learngit.git
```

After adding, the name of the remote repository is `origin`, which is the default name of Git. It can also be changed to something else, but the name `origin` is a remote repository at a glance.

> Push local files to remote repository

When pushing the local repository for the first time, you need to add the parameter -u, and the local `master` branch is associated with the remote new `master` branch

```bash
$ git push -u origin master
```

When you push the local repository next time, you only need to execute

```bash
$ git push origin master
```

> Unbind from remote repository

```bash
# First check the remote repository information
git remote -v

# Unbind remote repository by name
git remote rm origin
```

> Clone from remote repository

```bash
git clone git@github.com:michaelliao/gitskills.git
```

## Branch management

> Basic commands

View branches: `git branch`

Create branches: `git branch <name>`

Switch branches: `git checkout <name>` or `git switch <name>`

Create + switch branches: `git checkout -b <name>` or `git switch -c <name>`

Merge a branch into the current branch: `git merge <name>`

Delete a branch: `git branch -d <name>`

> Example

* Create `dev` branch

```bash
git branch dev

```

* Switch to `dev` branch

```bash
git checkout dev
```

The above two commands are equivalent to:

```bash
git checkout -b dev
```

* View the current branch

```bash
git branch
* dev
master
# Output all branches, the current branch is preceded by a *
```

Modify the readme.txt file in this branch and then commit the file

```bash
git add readme.txt
git commit -m "branch test"

# Switch back to the master branch
git checkout master
```

After switching back to the `master` branch, check a `readme.txt` file again, and the content just added is gone! Because that commit is on the `dev` branch, and the commit point of the `master` branch has not changed at this moment.

* Merge branches

```bash
git merge dev
```

The `git merge` command merges the specified branch into the current branch.

You can see the branch merge graph with the `git log --graph` command.

* Delete the dev branch

```bash
git branch -d dev
```

## Branch strategy for fixing bugs

Scenario: When you need to fix a bug, and the current work on dev has not been submitted, you cannot fix it by directly recreating a branch.

* Store unfinished work

Git provides a `stash` function that can "store" the current work site and continue working after restoring the site later.

```bash
$ git stash
Saved working directory and index state WIP on dev: f52c633 add merge
```

Now, use `git status` to view the working directory, which is clean (unless there are files not managed by Git), so you can safely create a branch to fix the bug.

* Create a temporary branch to fix the bug

```bash
# Switch back to the master branch
$ git checkout master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
(use "git push" to publish your local commits)

# Create a branch to fix the bug issue-101
$ git checkout -b issue-101
Switched to a new branch 'issue-101'
```

* Fix the bug and submit

```bash
$ git add readme.txt
$ git commit -m "fix bug 101"
[issue-101 4c805e2] fix bug 101
1 file changed, 1 insertion(+), 1 deletion(-)
```

* After fixing the bug, merge the branches and delete the branch issue-101

```bash
$ git switch master
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 6 commits.
(use "git push" to publish your local commits)

$ git merge --no-ff -m "merged bug fix 101" issue-101
Merge made by the 'recursive' strategy.
readme.txt | 2 +-
1 file changed, 1 insertion(+), 1 deletion(-)
```

* Switch back to the dev branch and restore the previous stash work site

```bash
$ git switch dev
Switched to branch 'dev'

# Check the workspace, it is clean, and you need to restore the previously saved work site
$ git status
On branch dev
nothing to commit, working tree clean

# Check the previous work site and find it stored somewhere
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge

# Restore the work site and continue development. At the same time, the previously saved work site will be deleted
$ git stash pop
On branch dev
Changes to be committed:
(use "git reset HEAD <file>..." to unstage)

new file: hello.py

Changes not staged for commit:
(use "git add <file>..." to update what will be committed)
(use "git checkout -- <file>..." to discard changes in working directory)

modified: readme.txt

Dropped refs/stash@{0} (5d677e2ee266f39ea296182fb2354265b91b3b2a)

# Or use the following command, which will not delete the previously saved stash work site
git stash apply
# If you need to delete the stash work site, use the following command
git stash pop
```

* View the stash work site

```bash
$ git stash list
```

* When there are multiple stashes, you can use the view command to restore the specified stash

```bash
$ git stash apply stash@{0}
```

* Bug fixes on the current dev branch

After fixing the bug on the master branch, we have to think about it. The dev branch was separated from the master branch in the early stage, so the bug actually exists on the current dev branch.

To fix the same bug on dev, we only need to "copy" the changes made by the commit `4c805e2 fix bug 101` to the dev branch. Note: We only want to copy the changes made by the commit `4c805e2 fix bug 101`, not merge the entire master branch.

For ease of operation, Git provides a special `cherry-pick` command that allows us to copy a specific commit to the current branch.

```bash
$ git branch
* dev
master
$ git cherry-pick 4c805e2
[master 1d4b803] fix bug 101
1 file changed, 1 insertion(+), 1 deletion(-)
```

## Multi-person collaboration

When you clone from a remote repository, Git actually automatically matches the local `master` branch with the remote `master` branch, and the default name of the remote repository is `origin`.

To view the information of the remote repository, use `git remote`:

```bash
$ git remote
origin
```

Or, use `git remote -v` to display more detailed information:

```bash
$ git remote -v
origin git@github.com:michaelliao/learngit.git (fetch)
origin git@github.com:michaelliao/learngit.git (push)
```

The above shows the address of `origin` that can be fetched and pushed. If you do not have push permissions, you will not see the push address.

> Push branch

Pushing a branch means pushing all local commits on the branch to the remote repository. When pushing, specify the local branch, so that Git will push the branch to the remote branch corresponding to the remote library:

```bash
$ git push origin master
```

If you want to push other branches, such as `dev`, change it to:

```bash
$ git push origin dev
```

However, it is not necessary to push the local branch to the remote, so which branches need to be pushed and which do not?

- `master` branch is the main branch, so it must be synchronized with the remote at all times;
- `dev` branch is a development branch, all team members need to work on it, so it also needs to be synchronized with the remote;
- `bug` branch is only used to fix bugs locally, so there is no need to push it to the remote, unless the boss wants to see how many bugs you fix every week;
- Whether the `feature` branch is pushed to the remote depends on whether you cooperate with your friends to develop on it.

> Grab branch

When multiple people collaborate, everyone will push their own changes to the `master` and `dev` branches.

Now, simulate your partner and clone it on another computer (note that you need to add the SSH key to GitHub) or in another directory on the same computer:

```bash
$ git clone git@github.com:michaelliao/learngit.git
Cloning into 'learngit'...
remote: Counting objects: 40, done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 40 (delta 14), reused 40 (delta 14), pack-reused 0
Receiving objects: 100% (40/40), done.
Resolving deltas: 100% (14/14), done.
```

When your partner clones from a remote repository, by default, your partner can only see the local `master` branch. If you don't believe it, you can use the `git branch` command to see:

```bash
$ git branch
* master
```

Now, if your friend wants to develop on the `dev` branch, he must create the `dev` branch of the remote `origin` locally, so he uses this command to create a local `dev` branch:

```bash
$ git checkout -b dev origin/dev
```

Now, he can continue to modify on `dev`, and then, from time to time, `push` the `dev` branch to the remote:

```bash
$ git add env.txt

$ git commit -m "add env"
[dev 7a5e5dd] add env
1 file changed, 1 insertion(+)
create mode 100644 env.txt

$ git push origin dev
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 308 bytes | 308.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
f52c633..7a5e5dd dev -> dev
```

Your friend has pushed his commits to the `origin/dev` branch, and you happened to make changes to the same file and tried to push:

```bash
$ cat env.txt
env

$ git add env.txt

$ git commit -m "add new env"
[dev 7bd91f1] add new env
1 file changed, 1 insertion(+)
create mode 100644 env.txt

$ git push origin dev
To github.com:michaelliao/learngit.git
! [rejected] dev -> dev (non-fast-forward)
error: failed to push some refs to 'git@github.com:michaelliao/learngit.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

The push failed because your partner's latest commit conflicts with the commit you are trying to push. The solution is also very simple. Git has prompted us to first use `git pull` to grab the latest commit from `origin/dev`, then merge locally, resolve the conflict, and then push:

```bash
$ git pull
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

git branch --set-upstream-to=origin/<branch> dev
```

`git pull` also failed because the link between the local `dev` branch and the remote `origin/dev` branch was not specified. According to the prompt, set the link between `dev` and `origin/dev`:

```bash
$ git branch --set-upstream-to=origin/dev dev
Branch 'dev' set up to track remote branch 'dev' from 'origin'.
```

Pull again:

```bash
$ git pull
Auto-merging env.txt
CONFLICT (add/add): Merge conflict in env.txt
Automatic merge failed; fix conflicts and then commit the result.
```

This time `git The pull is successful, but there are merge conflicts that need to be resolved manually. The solution is exactly the same as [Resolving Conflicts](http://www.liaoxuefeng.com/wiki/896043488029600/900004111093344) in branch management. After solving the problem, commit and push again:

```bash
$ git commit -m "fix env conflict"
[dev 57c53ab] fix env conflict

$ git push origin dev
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 621 bytes | 621.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0)
To github.com:michaelliao/learngit.git
7a5e5dd..57c53ab dev -> dev
```
Therefore, the working mode of multi-person collaboration is usually like this:

1. First, you can try to push your own changes with `git push origin <branch-name>`;

2. If the push fails, because the remote branch is newer than your local one, you need to use `git pull` to try to merge;
3. If there is a conflict in the merge, resolve the conflict and commit locally;
4. If there is no conflict or the conflict is resolved, push with `git push origin <branch-name>` and it will be successful!

If `git pull` prompts `no tracking information`, it means that the link relationship between the local branch and the remote branch has not been created. Use the command `git branch --set-upstream-to <branch-name> origin/<branch-name>`.

This is the working mode of multi-person collaboration. Once you are familiar with it, it is very simple.

> Summary

- View remote library information, use `git remote -v`;
- If the newly created local branch is not pushed to the remote, it will be invisible to others;
- Push the branch from the local, use `git push origin branch-name`, if the push fails, first use `git pull` to grab the new remote commit;
- Create a branch corresponding to the remote branch locally, use `git checkout -b branch-name origin/branch-name`, the names of the local and remote branches should be consistent;
- Establish the association between the local branch and the remote branch, use `git branch --set-upstream branch-name origin/branch-name h-name`;
- Get branches from the remote, use `git pull`, if there are conflicts, resolve the conflicts first.

## Tag Management

> Create tags

- The command `git tag <tagname>` is used to create a new tag, the default is `HEAD`, and you can also specify a commit id;
- The command `git tag -a <tagname> -m "blablabla..."` can specify tag information;
- The command `git tag` can view all tags.

It is very simple to tag in Git. First, switch to the branch that needs to be tagged:

```bash
$ git branch
* dev
master
$ git checkout master
Switched to branch 'master'
```

Then, type the command `git tag <name>` to add a new tag:

```bash
$ git tag v1.0
```

You can use the command `git tag` to view all tags:

```bash
$ git tag
v1.0
```

The default tag is the latest commit. Sometimes, if you forget to tag, for example, it is Friday now, but the tag that should have been added on Monday is not added, what should you do?

The method is to find the commit id of the historical commit and then type it in:

```bash
$ git log --pretty=oneline --abbrev-commit
12a631b (HEAD -> master, tag: v1.0, origin/master) merged bug fix 101
4c805e2 fix bug 101
e1e9c68 merge with no-ff
f52c633 add merge
cf810e4 conflict fixed
5dc6824 & simple
14096d0 AND simple
b17d20e branch test
d46f35e remove test.txt
b84166e add test.txt
519219b git tracks changes
e43a48b understand how stage works
1094adb append GPL
e475afc add distributed
eaadf4e wrote a readme file
```

For example, to `add Merge` commits this time and tags it. Its corresponding commit id is `f52c633`. Type the command:

```bash
$ git tag v0.9 f52c633
```

Use the command `git tag` to view the tag:

```bash
$ git tag
v0.9
v1.0
```

Note that the tags are not listed in chronological order, but in alphabetical order. You can use `git show <tagname>` to view tag information:

```bash
$ git show v0.9
commit f52c63349bc3c1593499807e5c8e972b82c8f286 (tag: v0.9)
Author: Michael Liao <askxuefeng@gmail.com>
Date: Fri May 18 21:56:54 2018 +0800

add merge

diff --git a/readme.txt b/readme.txt
...
```

You can see that `v0.9` is indeed included in the `add merge` commit.

You can also create tags with descriptions, specify the tag name with `-a`, and specify the description text with `-m`:

```bash
$ git tag -a v0.1 -m "version 0.1 released" 1094adb
```

You can see the description text with the command `git show <tagname>`:

```bash
$ git show v0.1
tag v0.1
Tagger: Michael Liao <askxuefeng@gmail.com>
Date: Fri May 18 22:48:43 2018 +0800

version 0.1 released

commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (tag: v0.1)
Author: Michael Liao <askxuefeng@gmail.com>
Date: Fri May 18 21:06:15 2018 +0800

append GPL

diff --git a/readme.txt b/readme.txt
...
```

> Tag Operations

- The command `git push origin <tagname>` can push a local tag;
- The command `git push origin --tags` can push all local tags that have not been pushed;
- The command `git tag -d <tagname>` can delete a local tag;
- The command `git push origin :refs/tags/<tagname>` can delete a remote tag.



