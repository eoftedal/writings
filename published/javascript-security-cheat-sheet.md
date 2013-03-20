Single Page Web App Security Cheat Sheet
========================================

This is intended as a simple abbreviated cheat sheet for securing javascript based single page web apps. It's not meant to cover everything in depth, but rather point you in the correct directions with resources.

What happens in JS, stays in JS
-------------------------------

Or to be more precise: What happens in the browser, stays in the browser. We cannot make security decisions for our data within the application. Any input validation (whether in html tags or implemented in JS) is purely cosmetic and for user guidance. Any security decisions based on data within the browser, needs to be double checked on the server. Anything happening in the browser can be altered. An attacker can access your services directly, thus circumventing any security implemented in the browser.

*Rule:* Access control, input validation and security decisions _must_ be made on the server.

Cover your XSS
--------------

Cross site scripting is a serious vulnerability. Even though XSS is often demonstrated using simple alert boxes, XSS is a common vector for delivering exploits. Consider using XSS to add an applet tag pointing to a malicious java application. Now we're getting serious.

We need to [handle untrusted data](http://erlend.oftedal.no/blog/?blogid=127) whenever we are outputting data and whenever we are building code from strings (eval, new Function, setTimeout, setInterval). When outputting data, we need to [be aware of context](https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet). Regarding building code from strings, just avoid it if you can.

And untrusted data can come from [so many places](http://code.google.com/p/domxsswiki/wiki/Sources). Some examples are URIs, JSON services, window.referrer, window.name, input fields, cookies.

*Rule:* Handle untrusted data with care - use contextual encoding and avoid building code from strings.

You have been served
--------------------

In a single page web app, private data only exists in the app's JSON services, thus it goes without saying we need to protect these services. We need to make sure [authentication](http://erlend.oftedal.no/blog/?blogid=128) and [authorization](http://erlend.oftedal.no/blog/?blogid=133) (access control) is properly implemented. We need to make sure we treat incoming data correctly - taking into account character sets, content-type, input validation. We need to make sure we don't expose unexpected data (e.g. a user's password hash). And we need to avoid [mass assigment](http://erlend.oftedal.no/blog/?blogid=129) - allowing an attacker to change fields we don't expect to be changed. We need to avoid things like [CSRF](http://erlend.oftedal.no/blog/?blogid=130).

*Rule:* Protect your services

Head's up
---------

Browsers these days support a number of HTTP headers we can use to protect our app. These include X-Frame-Options to avoid [Clickjacking](http://www.sectheory.com/clickjacking.htm), [Content Security Policy](https://developer.mozilla.org/en-US/docs/Security/CSP) to mitigate Cross Site Scripting and related attacks, [X-Content-Type-Options](http://msdn.microsoft.com/en-us/library/ie/gg622941%28v=vs.85%29.aspx) to make sure the browser cannot be tricked into for instance [interpreting JSON as HTML](http://erlend.oftedal.no/blog/research/json/testbench.html) (only works in some browsers).

*Rule:* Learn how to use security HTTP headers

Resources
---------

* [OWASP Top 10 for JavaScript](http://erlend.oftedal.no/blog/?blogid=125)
* [DOM XSS Wiki](http://code.google.com/p/domxsswiki/wiki/Sources)
* [DOM Based XSS Prevention Cheat Sheet](https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet)
* [Content Security Policy](https://developer.mozilla.org/en-US/docs/Security/CSP)
* [Clickjacking](http://www.sectheory.com/clickjacking.htm)

Suggestions?
------------

Got ideas for improving this cheat sheet? Send me an [email](mailto:erlend@oftedal.no) or send me a pull request.
