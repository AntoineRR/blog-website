---
layout: article
title: Git cheat sheet - Commands I use every day
mathjax: false
show_subscribe: false
---

As a software development engineer, I obviously use git **a lot**. I always preferred to use the command line for git as it is a knowledge that does not depend on a specific GUI application or IDE. Commits after commits, I realized I often use the same commands over and over again and decided to share the most useful ones here.

<!--more-->

I won't be going over what is git and why you should use it, but only share some handy command lines :slightly_smiling_face:
{:.info}

# Add, commit, push!

After you did your changes, you probably want to commit and push them on the remote server. This is where the git *holy trinity* finds its use:

```console
$ git add *
```
*to add all files*

```console
$ git commit -m "<put your commit message here>"
```
*to commit your changes*

```console
$ git push
```
*to push your changes on the server*

It's as simple as that! With those three commands, all the changes you made are now saved on your remote server!

You will first have to configure your remote server, such as Github or Gitlab. To do this, you should create a repository on the website and follow the guidelines they describe to push your initial commit.
{:.info}

To choose a useful commit message, I like to follow the [conventional commit](https://www.conventionalcommits.org/en/v1.0.0/#summary) guidelines.
{:.info}

Those commands have more to offer, and I also use them in different *flavors*. For example:
- You can add only a subset of the files you modified by specifying them: `git add file1 file2 file3 ...` instead of adding everything. The files you didn't add will not be committed.
- You can add and commit your modified files in one command using `git commit -am "<put your commit message here>"`. However, this will not add the new files, only the files you modified! I still use the `-a` flag a lot when I only modified files, because I can skip the `git add` command in this case (being lazy can save time :slightly_smiling_face:).
- You can use git push only if your branch exists on the remote server. If it doesn't exists because you created it on your local repository, you can push it to the remote server after committing your code using `git push --set-upstream <branch-name>`.

# Oh no, I messed up my commit history...

Good, you committed your changes and pushed them online! Now, your [CI/CD](https://en.wikipedia.org/wiki/CI/CD) pipeline with all the unit tests runs and... ***Ouch***. Some tests are failing. You made a mistake and have to do some changes on your code to fix them.

Ok, no big deal, you write a fix on your local project, commit it and push it again online. The tests are now passing! But... You added a new commit that is not really useful, and it just leaves a *noisy* commit history. If you are like me and like to have a clean history to remember what commit had what effect, this is annoying. How can we avoid such issue?

Be very careful when rewriting your commit history! It is a powerful yet dangerous thing to do (you probably don't want to end up dropping a commit or merging your commit with one from someone else)!
{:.error}

## Unlimited power: interactive rebase

To clean this now messy history, we can use an interactive rebase:
```console
$ git rebase -i HEAD~2
```

This command will open your default text editor and show you the last 2 commits you did, prefixed by the `pick` keyword. The latest commit is the last one.
```
pick b4dcb59 feat: does something
pick ece6408 fix: tests
```

You can view more than 2 commits by modifying the number after the `~` in the command.
{:.info}

To merge the second commit with the first one, remove the `pick` in front of the second commit and replace it with `fixup`.
```
pick b4dcb59 feat: does something
fixup ece6408 fix: tests
```

Then save and close the file (`Ctrl+X` in nano editor, `:wq` for vim). You will have a single commit as a result, with the message from the first commit (the message of the second commit will be dropped)! The commit history is now much cleaner, and it appears as if you didn't make a mistake at all (what a genius)!

There are other keywords you can use instead of `fixup`, those are detailed in the file you have to modify when running the command, in the comments below your commits.
{:.info}

## Gotta go fast: commit amend

We are now done, we fixed our commit! However, the process was quite long and error prone, especially when we modified the file. This is where new flags become useful! When you want to merge your current changes with the last commit you did, simply run:

```console
$ git commit --amend --no-edit -a
```

This magic command will automatically merge your current changes into the previous commit, without changing the commit message! the `-a` flag even adds your changes without `git add` as discussed previously.

Depending on your needs, you may either use an interactive rebase or a commit amend. Most of the time, a commit amend is enough for me, but interactive rebase do allow for more modifications, such as dropping commits you don't need anymore for example.

## Push it!

Alright, now that we are back to a single commit including the fix, let's push it to the remote server using `git push`!

... Why doesn't it work?

Oh! It won't let us push because we changed our commit history...

In fact, in this situation, we want to override the previous *bad* commit (on the server) with our new *fixed* one (on our local machine). We have to force push our commit to the remote branch:
```console
$ git push origin +<branch-name>
```

The `+` sign in front of the branch name means we are force pushing! This is a really nice shortcut I often use to fix my commits that where pushed online.

As you did some modifications to the commit history, check if you got the expected result (with `git log`) before force pushing your changes online! The online changes will be overwritten with your local ones!
{:.error}

# From branch to branch

I sometimes work on several features in parallel, and develop each of them on their own git branch. I create those branches using:
```console
$ git checkout -b <branch-name>
```

The `-b` flag creates the branch and the `checkout` command makes you move to this new branch. I can now do some changes to the files, and I sometimes have to move to another branch while my work is not done on the branch I am on! This is annoying because git will not let you `git checkout <branch-name>` while you have uncommitted changes on your current branch. You could then commit your changes with a temporary `wip` commit you will modify later, but I do not really like this solution (remember I like clean commit histories?). One thing I always do in this case is:
```console
$ git stash
```

This will store all your changes in a "stash", and leave you with a clean working tree. In case you had new files, these must be added with `git add` before for them to end up in your stash. The drawback of this method is that I sometimes have several stashes and do not remember what they correspond to. If you want to add some useful information for your future self to your stash, simply add `save` followed by your message:
```console
$ git stash save <message to remember what I was doing>
```

Once you did this, you can checkout to the branch you wanted, do whatever you had to and come back to the previous branch. Your stashed changes can then be applied again by running:
```console
$ git stash pop
```

This will apply the changes from the **last** stash you did. But what if you stashed several things and want to apply a stash you did a long time ago? You will then need to list your stashes, and find the one you want to apply. Unsurprisingly, there is a command for doing exactly this:
```console
$ git stash list
```

This is where having saved your stash with a special message becomes useful, it will be displayed near the corresponding stash. It is then easier to find in the list! The important thing to remember about the stash you want to apply is its number. All you have to do to apply it afterwards is:
```console
$ git stash pop <number>
```

Note that the `pop` command will apply *and* delete your stash. If you want to apply the stash and keep it in your stash list, use `git stash apply`.
{:.info}

# Bonus commands!

```console
$ git status
```
I use it all the time before making any commit, to make sur I added the files I want. This will log the new/modified/deleted files, in green for the ones you added with `git add` and red for the one you didn't add.

```console
$ git log
```
This command will log the commits of the project, latest first. Simply type `q` to stop browsing the commit history. It can be useful to make sure you will `git commit --amend` the right commit and to check if an interactive rebase (`git rebase -i`) did what you wanted.
