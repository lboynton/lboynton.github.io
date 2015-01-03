---
layout: post
title: Trying Out PHP in NetBeans 6.5
date: 2008-12-05 23:48:22.000000000 +00:00
categories: web-dev
tags:
- dreamweaver
- NetBeans
- open source
- php
permalink: /2008/12/05/trying-out-php-in-netbeans-65/
---
I have been trying out NetBeans 6.5's PHP support in recent weeks, whereas previously I was mostly using Dreamweaver. Here are some things I think NetBeans does better than Dreamweaver (in CS3, I haven't tried CS4):

*   OOP support. Although it's not as good as it is for Java. For instance, when subclassing an abstract class NetBeans will not provide me with a way to override the abstract methods for it. Anyone know why? Instead I have to copy and paste the abstract methods which is quite tedious and can be error prone. It also does not show errors when I try to access a variable which does not exist or call methods which do not exist. NetBeans knows when a method does not exist because it won't go to the method declaration when ctrl+clicking the method name.
*   Support for PHPdoc. NetBeans will create a skeleton PHPdoc block as soon as you type /** and hit enter, this works the same as for Java.
*   NetBeans stores project specific files along with the project itself. This means the project is much more portable between different systems (as long as those systems run NetBeans). The project files are included in any version control systems so you can quickly open the project on another system. In Dreamweaver you have to create a new site on each computer or manually export/import the site definitions.
*   NetBeans will highlight all references to a variable when you place the cursor inside the variable name.
*   NetBeans allows you to rename variables and functions once, and it will update it everywhere else. OK, this doesn't work quite how I'd like. For instance it only seems to update the references to the functions or variables in the same file.
*   PHP debugger. I haven't used this yet.
*   It's easy to go to the function declaration you are looking for in a class using NetBeans' navigator which will take you to the line the function starts at by double clicking its name in the function list (which is helpfully alphabetically sorted).
*   Database integration. I haven't used this much, but it allows you to communicate with the database without an external tool. There appear to be drivers for at least MySQL and PostgreSQL.
*   Support for different version control systems. I believe CS4 only has support for Subversion, but NetBeans supports CVS, SVN, Mercurial and probably others.
*   Support for JavaScript frameworks. Haven't used this yet, but apparently NetBeans will provide code completion and integrated documentation for Prototype, Scriptaculous, JQuery, Dojo etc. This should be very useful as it means no more trips to the API website.

Of course, this is aside from the fact NetBeans is open source, much more extensible and configurable, and is cross-platform. On the other side, Dreamweaver is a nicer environment to work in visually. Also:

*   Dreamweaver's FTP functionality is much more mature than NetBeans. For example, NetBeans does not support SFTP, and synchronising is easier in Dreamweaver.
*   NetBeans does not seem to support CSS code completion, a feature I really miss as it's impossible to know every CSS property. NetBeans does however offer information about the values that can be applied to each CSS property and a preview window which is kind of useful.

One thing which took me a while to work out:

*   Because PHP doesn't use object references, you have to add the return types to the PHPdoc so that NetBeans knows what object types functions return for the code completion to work.

I think what NetBeans needs to work on a bit is polish; make it a more appealing environment to work in. I would like to see NetBeans have support for PHP frameworks such as CakePHP and Zend Framework in the future as well. One of these days I intend to try out some other IDEs, such as Eclipse. For the time being though I shall continue developing PHP websites in NetBeans.