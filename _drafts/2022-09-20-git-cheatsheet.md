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

