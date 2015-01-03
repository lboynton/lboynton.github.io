---
layout: post
title: Debugging ejabberd
date: 2011-06-16 22:42:14.000000000 +01:00
categories: programming
tags:
- breakpoints
- debug
- ejabberd
- erlang
- step through
permalink: /2011/06/16/debugging-ejabberd/
---

This took me a little while to work out, and will probably be useful in the future so I thought I'd make a post about it and use my blog for the first time in a year and a half ;)

1. Download the ejabberd source:

	<pre>git clone https://github.com/processone/ejabberd.git</pre>

2. Configure (I want ODBC support, but you might not need it)

	<pre>cd ejabberd &amp;&amp; ./configure --enable-odbc</pre>

3.  Compile code with debugging enabled

	<pre>make debug=true</pre>

4.  Install ejabberd; I couldn't work out if there was a way to run ejabberd without installing it.

	<pre>sudo make install</pre>

5.  Configure ejabberd as required by editing /etc/ejabberd/ejabberd.cfg. Then run it with

	<pre>sudo ejabberdctl start</pre>

6.  Attach to the running erlang process

	<pre>sudo ejabberdctl debug</pre>

7.  Press any key to continue. Then in the shell type:

	<pre>im().</pre>

This will open up the lovely erlang debugger monitor window, as shown below.

![Erlang debugger](/assets/Screenshot-Monitor-1.png "How pretty it is!")

Then go to Module -&gt; Interpret and browse to the ejabberd source code folder from earlier, and choose the .erl file you want to debug. This only works if the beam files have been compiled with debugging enabled, using the make step from before. Once you've selected a file, it will appear in the list on the left and you can double click it to view it and add break points as shown below.

![mod_muc_room](/assets/Screenshot-View-Module-mod_muc_room.png "Adding a function breakpoint")

To trigger the break point you just need to join a chat room with a client:

[caption id="attachment_216" align="aligncenter" width="583" caption="Break point triggered"][![Break point triggered](/assets/Screenshot-Monitor.png "Break point triggered")](http://lboynton.com/wp-content/uploads/2011/06/Screenshot-Monitor.png)[/caption]

You can then step through the code by double clicking on the line with the break in it:

[caption id="attachment_217" align="aligncenter" width="337" caption="Step through debugging"][![Step through debugging](/assets/Screenshot-Attach-Process-0.614.0.png "Step through debugging")](http://lboynton.com/wp-content/uploads/2011/06/Screenshot-Attach-Process-0.614.0.png)[/caption]

Happy debugging!