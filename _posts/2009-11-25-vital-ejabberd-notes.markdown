---
layout: post
title: Vital ejabberd Notes
date: 2009-11-25 11:37:12.000000000 +00:00
categories: sys-admin
tags:
- ejabberd
- frustration
permalink: /2009/11/25/vital-ejabberd-notes/
---
# Making Changes Stick

Put

> override_global.
>
> override_local.
>
> override_acls.

At the top of the ejabberd.cfg file to ensure that changes in the configuration file take affect. Otherwise, you may experience strange results where changes do not seem to take affect after restarting the server. This is because the configuration file is loaded into the internal ejabberd database. If you do not override certain settings in the configuration file, the old values from the database will be used. Spent much time trying to work out why my changes to the configuration file weren't being noticed.

# Controlling ejabberd

Erlang, the language ejabberd is written in, uses nodes. These are controlled with ejabberdctl. ejabberd itself can be controlled with the init script, /etc/init.d/ejabberd. If you get:

> Node 'ejabberd@host' is started. Status: started
>
> ejabberd is not running

That means the node is running, but inside that ejabberd didn't start correctly, but didn't crash. Usually this is a misconfiguration, and the error message will be in a log file, either /var/log/ejabberd/ejabberd.log or /var/log/ejabberd/sasl.log. When making changes to the configuration file, usually you just use ejabberdctl restart, and don't use the init script.
