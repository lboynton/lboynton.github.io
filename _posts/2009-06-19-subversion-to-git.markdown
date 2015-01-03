---
layout: post
title: Subversion to Git
date: 2009-06-19 11:39:10.000000000 +01:00
categories: programming
tags:
- git
- git push
- subversion
- svn
- version control
permalink: /2009/06/19/subversion-to-git/
---
Recently I decided to have a look at using Git for my projects instead of Subversion. Git has many advantages over Subversion, the main one I was interested in is that it is a distributed VCS. This has the benefit that code can be committed and branched in your own local repository and it does not rely on network access. The only downside I can see to this is that you are perhaps less likely to have a backup of your code on a separate machine. Therefore I was interested in setting up Git so that I can still commit my changes to a centralised location for a backup. As a result, this post is intended to serve as a quick introduction to Git and a how-to for setting up a repository for pushing changes to.

# Git usage

Set the name and email address which your commits will use with:

<pre>git config --global user.name "Your name"
git config --global user.email "Your email"</pre>

This gets stored in ~/.gitconfig. Removing the --global flag will apply the values to a specific project if in a git project directory.

To initialise a new git repository, go to the root directory of the project files and use:

<pre>git init</pre>

To add files to be committed use the following. This applies to both new files and modified files which should be committed, which differs to Subversion in which only new files need to be added and all modified files will be automatically added.

<pre>git add file.txt</pre>

To remove a file which has been added to the next commit:

<pre>git reset HEAD file.txt</pre>

To delete a file from the project, which will be removed during the next commit:

<pre>git rm file.txt</pre>

To diff the files which have not been added to the next commit:

<pre>git diff</pre>

To diff the files which have been added to the next commit:

<pre>git diff --cached</pre>

To commit changes to the repository:

<pre>git commit</pre>

To commit changes to the repository and automatically add any modified (but not new) files:

<pre>git commit -a</pre>

To view the log of all previous commits:

<pre>git log</pre>

To get a copy of a Git project:

<pre>git clone /path/to/project.git</pre>

# Setting up a push repository

For this I found [this link](http://toolmantim.com/articles/setting_up_a_new_remote_git_repository) to be useful. Essentially you connect to the server you wish to push changes to via SSH or some other method, create a directory for the project and initialise it as a bare repository.

<pre>git --bare init /path/to/project.git</pre>

And then on the system you wish to push from, add a remote repository, in the following case it will be called "origin":

<pre>git remote add origin ssh://myserver.com/path/to/project.git</pre>

Then pushes can be performed using:

<pre>git push origin master</pre>

It is important not to push to a repository with a working copy, as changes may get lost.

# Resources

The following resources were used:

*   [Git book](http://book.git-scm.com/ "Git book")
*   [Setting up a new remote git repository](http://toolmantim.com/articles/setting_up_a_new_remote_git_repository "Setting up a new remote git repository")
