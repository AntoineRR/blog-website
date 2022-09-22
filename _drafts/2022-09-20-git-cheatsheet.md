---
layout: article
title: Git cheat sheet - Commands I use every day
mathjax: false
show_subscribe: false
---

As a software development engineer, I obviously use git **a lot**. I always preferred to use the command line for git as it is a knowledge that does not depend on a specific GUI application or IDE. Commits after commits, I realized I often use the same commands over and over again and decided to share the most important ones here.

<!--more-->

I won't be going over what is git and why using it here, but only share some handy command lines :slightly_smiling_face:
{:.info}

# Where it all begins...

After you did your changes, you probably want to commit them, and push them on the remote server. This is where the git holy trinity finds its use:

```
$ git add *
```
*to add all files*

```
$ git commit -m "<put your commit message here>"
```
*to commit your changes*

```
$ git push
```
*to push your changes on the server*

It's as simple as that! With those three commands, all the changes you made are now saved on your remote server!

For more information:
- You can add only a subset of the files you modified by specifying them: `git add file1 file2 file3 ...` instead of adding everything.
- You can add and commit you modified files in one command using `git commit -am "<put your commit message here>"`. This will not add the new files, only the files you modified!
- You can use git push only if your branch exists on the remote server, more on that on the next section.

# After pushing online

Good, you commited your changes and pushed them online! Now, your CI/CD pipeline runs and... *Ouch*. Some tests are failing. You made a mistake and have to do some changes for your code to work.

Ok, no big deal, you write a fix on your local project, commit it and push it again online. The tests are now passing again! But... You added a new commit that is not really useful, and it just leaves a "noisy" commit history. If you are like me and like to have a clean history to remember what commit had what effect, this is annoying. How can we avoid such issue?

To clean this now messy history, we will need an interactive rebase:
```
git rebase -i HEAD~2
```

This command will open your default text editor and show you the last two commits you did, prefixed by the `pick` keyword. To merge the second commit with the first one, remove the `pick` in front of the second commit and replace it with `fixup`, then save the file. You will have a single commit as a result, with the message from the first commit (the message of the second commit will be droped).

Alright, now that we are back to a single commit including the fix, let's push it to the remote server using `git push`! Oh... It won't let us push because our local tree is now different from the remote tree...

In fact, in this situation, we want to override the previous "bad" commit with our new "fixed" one. We have to force push our commit to the remote branch:
```
git push origin +<branch_name>
```

The `+` sign in front of the branch name means we are force pushing! This is a really nice shortcut I often use to fix my commits that where pushed online.

We are now done, we fixed our commit! However, the process was quite long and error prone, especially with the interactive rebase. This is where new flags become useful! When you want to merge your current changes with the last commit you did, simply run:

```
git commit --amend --no-edit -a
```

This magic command will automatically merge your current changes into the previous commit, without changing the commit message!

# In case of a mistake...

```
git reset
```

# Some other useful commands

```
git status
```

```
git log
```
