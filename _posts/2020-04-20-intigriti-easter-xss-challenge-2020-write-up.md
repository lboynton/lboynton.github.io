---
title: Intigriti Easter XSS Challenge 2020 Write Up
layout: post
---

I managed to solve the recent [Intigriti Easter XSS challenge](https://challenge.intigriti.io/) with less than 1 hour to spare! Here is my write up.

# The Challenge
The challenge was to display an alert box on the page with the value of `document.domain`. Here is a screenshot showing the rules:

![XSS challenge rules](/assets/intigriti-easter-xss-challenge-rules.png)

There is a small demo app. When you select one of the dropdown options, an XHR request is made in the background to display more information. The location hash changes to #1, #2, #3 etc depending on which item in the select box you select.

![Example of location.hash](/assets/intigriti-challenge.png)

# First Thoughts
I first had a look at the source of the page to see what JavaScript was being loaded. There was only one JS file called source.js. The contents are as follows:

```javascript
var hash = document.location.hash.substr(1);
if(hash){
  displayReason(hash);
}
document.getElementById("reasons").onchange = function(e){
  if(e.target.value != "")
    displayReason(e.target.value);
}
function reasonLoaded () {
    var reason = document.getElementById("reason");
    reason.innerHTML = unescape(this.responseText);
}
function displayReason(reason){
  window.location.hash = reason;
  var xhr = new XMLHttpRequest();
  xhr.addEventListener("load", reasonLoaded);
  xhr.open("GET",`./reasons/${reason}.txt`);
  xhr.send();
}
```
The first thing that stood out immediately was the use of the [`Element.innerHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) property. This is notorious for being an XSS vector and is often used instead of the safe alternative of [`Node.textContent`](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent).  Any HTML being passed in to the property will be rendered, including script tags.

I decided the goal was obviously to inject some of our own JavaScript here, which means `this.responseText` needs to contain our payload. I also noticed that `unescape()` is being used so our payload needs to be URL encoded. Looking at the code I could see that `this.responseText` was being set as a result of the load event being triggered once the XHR request to `./reasons/${reason}.txt`. I could also see that the location hash was being used to trigger the XHR as well the change event handler on the select box. This means that the payload could be passed through the hash in the URL, and the alert should popup by clicking on a carefully crafted link.

# Causing Errors
Knowing that the location hash triggered XHR requests, I tried loading <https://challenge.intigriti.io/#foobar> to see what happens when an unexpected value is passed in through the hash.

![Passing foobar in location.hash](/assets/intigriti-foobar.png)

Great I thought! It reflects whatever value you pass in to `location.hash`, so I can just pass in my XSS payload:

![First attempt](/assets/intigriti-failed-attempt1.png)

Oh :( Sadly this doesn't work because it seems the payload is being returned [URL encoded ](https://en.wikipedia.org/wiki/Percent-encoding) but with the percentages replaced with underscores (%3C is a URL encoded `<`) . I tried all sorts of things to try to get a percent sign to be returned in the body of the 404 response, but I was hitting my head against the wall and decided to call it a day.
# Tips
The Intigriti twitter feed provided many tips along the way for how to solve the challenge. The one that helped me breakthrough was the last tip:

> 900 likes, time for the last hint: 403+404=1337

Ah ha! I hadn't considered there might be other errors we can try to trigger to get our payload reflected in the response. I started thinking about what could cause a 403, and immediately thought of `.htaccess` files as apache is normally configured to deny direct access to these files. I tried accessing <https://challenge.intigriti.io/.htaccess> and sure enough a 403 response was returned, and the filename was being reflected in the response. This is a standard apache 403 response.

I tried adding a query string parameter to see if that was also reflected in the response, which it was.

Remembering that the response is URL decoded before being passed to `innerHTML`, I knew that I needed to URL encode the payload. The browser already applies URL encoding to the request path, so you need to double URL encode it to get back a single URL encoded value in the response. I used PHP to do this:

```
php > echo rawurlencode(rawurlencode('<img src=foo onerror=alert(1)>'));
%253Cimg%2520src%253Dfoo%2520onerror%253Dalert%25281%2529%253E
```

![Blocked by CSP](/assets/intigriti-blocked-by-csp.png)

The HTML is rendered as shown by the broken image, but the onerror event handler did not trigger.
# Bypassing CSP
I was expecting a CSP bypass to be part of the challenge as it was mentioned in the rules, but was confident it could be bypassed. However, after reviewing the rule:

> content-security-policy: default-src 'self'

I become less confident. This is quite a strict rule, as [Google's CSP](https://csp-evaluator.withgoogle.com/) evaluator shows:

![](/assets/intigriti-csp-evaluator.png)

No glaring issues here. I figured I would have to use a reflected request to load a JavaScript file into the running page, to satisfy the self CSP rule. I also decided this is what the 404 response would be for.

# Returning JavaScript through a 404
My next attempt to get some JavaScript to run was to use the 403 to return a response with a script tag that has a src attribute pointing to a 404 page with some code reflected in the response. This looked like the following:

```
php > echo rawurlencode(rawurlencode('<script src="/document.domain"></script>'));
%253Cscript%2520src%253D%2522%252Fdocument.domain%2522%253E%253C%252Fscript%253E
```

This does not even attempt to load the 404 page because [according to MDN](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML#Security_considerations):

> HTML5 specifies that a <script> tag inserted with innerHTML should not execute.
	
This means we need to find another way of loading a script onto the page. I thought about using an iframe to embed a script tag in, as this would create a new DOM and thought this would probably bypass the innerHTML restriction. The iframe only needed to contain the script tag, so I looked for ways of embedding HTML directly in an iframe without using a `src` attribute. Some Googling led me to the `srcdoc` attribute.

# Using srcdoc

My next attempt was as follows:
	
```
php > echo rawurlencode(rawurlencode('<iframe srcdoc="<script src=/alert(document.domain)></script>"></iframe>'));
%253Ciframe%2520srcdoc%253D%2522%253Cscript%2520src%253D%252Falert%2528document.domain%2529%253E%253C%252Fscript%253E%2522%253E%253C%252Fiframe%253E
```

Appending this URL encoded string after `.htaccess?` resulted in the following:

![First iframe attempt]({{ 'assets/intigriti-first-iframe-attempt.png' | relative_url }})
	
This looked good, although I knew it wasn't going to trigger the alert yet. I could see that it was making a request for the 404 page now, and it was returning my `alert(document.domain` payload. I knew now I just needed to make the response syntactically valid JavaScript.
	
# Making valid JavaScript

The response from my attempt was:
	
```javascript
404 - 'File "alert(document.domain)" was not found in this folder.'
```
	
This isn't valid JavaScript. To make it valid you need to split it into three parts. By putting a single quote and a semicolon before the alert, you make the first part valid. By adding a semicolon and a single quote after the alert, you make two more valid blocks of JavaScript. The response needs to look as follows:
	
```javascript
404 - 'File "';alert(document.domain);'" was not found in this folder.'
```

My next attempt was as follows:
	
```
php > echo rawurlencode(rawurlencode('<iframe srcdoc="<script src=/\';alert(document.domain);\'></script>"></iframe>'));
%253Ciframe%2520srcdoc%253D%2522%253Cscript%2520src%253D%252F%2527%253Balert%2528document.domain%2529%253B%2527%253E%253C%252Fscript%253E%2522%253E%253C%252Fiframe%253E
```
	
Crossing my fingers I pasted this URL encoded string after the .htaccess? in the URL, and voilà:

![Successful XSS]({{ 'assets/intigriti-success.png' | relative_url }})

Success!
