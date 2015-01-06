---
layout: post
title: Unit Testing Zend_Http_Client Based Web Service Calls
date: 2012-09-18 19:39:14.000000000 +01:00
categories: web-dev
tags:
- php
- phpunit
- unit testing
permalink: /2012/09/18/unit-testing-zend_http_client-based-web-service-calls/
---
This post has been sitting as a draft for ages (since before Zend Framework 2.0 ;)) but I thought I should get it out anyway!

When testing code which makes HTTP requests to web services, you do not want your tests to be reliant on the web service working. This goes against the best practices of unit tests, in which tests should be quick to run (web service might be slow), tests should ideally require no set up to run (web service calls may need API keys) and tests should be as standalone as possible (so that they can be run on say, a continuous integration server, or so you can test offline!). The point of this is so that you have no excuse not to run the tests whenever you make a change to the code. The idea of running unit tests on the client is also not to find bugs in the web service itself.

Thus, here is a basic example of a web service client class and it's corresponding test class. By using the Zend_Http_Client_Adapter_Test adapter, we can tell Zend_Http_Client when we want it to fail, and test that the code handles failure gracefully.

{% gist lboynton/3744912 %}