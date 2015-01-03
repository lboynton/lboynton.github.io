---
layout: post
title: NetBeans 6.7 and PHP License Templates
date: 2009-07-19 16:28:31.000000000 +01:00
categories: web-dev
tags:
- NetBeans
permalink: /2009/07/19/netbeans-6-7-and-php-license-templates/
---
It seems there is a bug in NetBeans 6.7 which means that PHP projects do not work correctly with license templates. If you specify a license to use in the nbproject/project.properties file, the template file does not seem to read that key, and thus the default template is used. This has been fixed in the [daily builds](http://bits.netbeans.org/dev/nightly/), bug report is [here](http://www.netbeans.org/issues/show_bug.cgi?id=167661).
