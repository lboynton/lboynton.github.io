---
layout: post
title: Git - Ignoring changes in previously committed files
date: 2009-07-12 22:56:17.000000000 +01:00
categories: programming
tags:
- git
- version control
permalink: /2009/07/12/git-ignoring-changes-in-previously-committed-files/
---
As the .gitignore file only applies to files which aren't being tracked, and aren't in the repository, an alternative method must be used to ignore changes to files which are in the repository:

<pre>git update-index --assume-unchanged &lt;file&gt;</pre>

Note that the file can still be added to the commit explicity with git add, and this change does not get committed - ie the file won't be ignored for other clones.
