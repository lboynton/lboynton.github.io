---
layout: post
title: Zend_OpenId_Provider_Storage_Db?
date: 2008-12-01 01:45:01.000000000 +00:00
categories: web-dev
tags:
- openid
- php
- web dev
- zend framework
permalink: /2008/12/01/zend_openid_provider_storage_db/
---
Recently I have been working on a project to create an OpenID provider using the Zend Framework. I was somewhat disappointed upon finding out that there was only the one storage method available - file based storage. Files are stored only temporarily too. Now how useful is this? It's fine for a quick demonstration of how to setup OpenID, but beyond that it does not serve much purpose if you ask me. Storing user accounts in the database is much more common. So why can this not be included in the core Zend Framework library?

Does anyone have a database storage class they would mind sharing? Also, is there a site similar to CakePHP's bakery for Zend Framework? The bakery is useful as it allows the community to share their code, and not be restricted to just the core functionality of the framework. I am only about a month or so into my journey into learning the Zend Framework, but so far I definitely prefer Cake.
