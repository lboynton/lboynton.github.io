---
layout: post
title: ejabberd mod_archive with MySQL on Ubuntu
date: 2009-11-25 12:57:21.000000000 +00:00
categories: sys-admin
tags:
- ejabberd
- mod_archive
- mod_archive_webviewer
- mysql
- odbc
permalink: /2009/11/25/ejabberd-mod_archive-with-mysql-on-ubuntu/
---
1.  Install the MySQL ODBC driver,

    > apt-get install libmyodbc

2.  Create an ODBC configuration for ejabberd at /etc/odbc.ini,
<pre class="php">[ODBC Data Sources]
odbcname     = MyODBC 3.51 Driver DSN

    [ejabberd]
Driver       = /usr/lib/odbc/libmyodbc.so
Description  = MyODBC 3.51 Driver DSN
SERVER       = localhost
PORT         =
USER         = ejabberd
Password     = ejabberd
Database     = ejabberd
OPTION       = 3
SOCKET       =</pre>

3.  Download mod_archive from Subversion.

<pre>svn co https://svn.process-one.net/ejabberd-modules
cd ejabberd-modules/mod_archive/trunk</pre>4.  You may then have to edit Emakefile before compiling. This is because by default it will point to the include files from ejabberd trunk, which will probably be for a newer version of ejabberd than you have installed, unless you installed ejabberd from trunk. in Emakefile replace all instances of:

<pre>trunk</pre>

With

<pre>branches/ejabberd-2.0.x</pre>

And then run

<pre>./build.sh</pre>5.  This will generate some *.beam files in the ebin directory. Copy these to the ejabberd ebin directory which should contain all the other ejabberd .beam files. The directory for me is /usr/lib/ejabberd/ebin/.
6.  Create the MySQL database schema using the SQL scripts from mod_archive. They are in ejabberd-modules/mod_archive/trunk/src/*.sql. The script for MySQL is mod_archive_odbc_mysql.sql.
7.  Add in the configuration options to enable mod_archive in ejabberd.cfg, under modules:
8.  <pre>{mod_archive_odbc, []},</pre>

9.  Create ODBC connection:

<pre>{odbc_server, "DSN=ejabberd;UID=ejabberd;PWD=ejabberd"}.</pre>

Note that the authentication won't use this by default because in the configuration file there is an option:

<pre>{auth_method, internal}.</pre>

Which means it will use the internal mnesia database for auth.10.  Finally, to view the archives in a web browser, add to the listen section of the ejabberd.cfg file:
<pre>{request_handlers, [{["archive"],mod_archive_webview}]}</pre>

So that the final section for port 5280 looks something like

    {5280, ejabberd_http, [
       http_poll,
       web_admin,
                         {request_handlers, [{["archive"],mod_archive_webview}]}
      ]}

     ]}.

11.  The archives should then be visible at http://host:5280/archive/

## More info

*   [https://help.ubuntu.com/community/ODBC](https://help.ubuntu.com/community/ODBC)