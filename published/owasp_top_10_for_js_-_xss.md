Cross site Scripting - or XSS - is probably one of the most common and one of the most difficult problems to fully mitigate. At first it seems simple, but as contexts grow in complexity and the amount of code grows, it get's harder to discover all the different sinks.

This is the risk rating from OWASP: 
![OWASP Risk rating XSS](http://beta.open.bekk.no/Home/Attachment/6e1ebb35-35b3-4137-8f9b-8ff986d3090c)

# Traditional XSS
We traditionally talk about two types of XSS - reflected and stored. In reflected XSS, the attack is a part of the URL like this one: `http://www.insecurelabs.org/Search.aspx?query="><script>alert(1)</script>` You can test that one [here](http://www.insecurelabs.org/Search.aspx?query=%22%3E%3Cscript%3Ealert%281%29%3C/script%3E). In reflected XSS, the attacker has to trick the user into opening the URL somehow - typically by employing iframes, phishing or shortened URLs.

In stored, or persistent, XSS, the attacker is able to store the attack string in the database. Since the attack is not dependent on the exact URL being visited anymore, the attacker can simply wait for a victim to visit the otherwise legit page. An example could be the commenting section here: [http://www.insecurelabs.org/Talk/Details/5].

# Twists on traditional XSS
Single page webapps are usually loading a static HTML-layout, which is the exact same file for all users, and then data and private information is loaded using JSON services. If there is HTML-tags inside our JSON-data we would normally expect the browsers not to render this content, but rather treat application/json as, well, JSON. However this is not always the case. Some browser, and especially IE, tends to try to second guess the content-type given by the server. This is called content-sniffing, and is in place because at some point in time servers tended not to send the correct content-type, and thus browsers could help the developers and end user by checking "what the content really was".

I built a small [test bed](http://erlend.oftedal.no/blog/research/json/testbench.html) for showing how this works. You will see a green check if the browser treats the content as HTML (and thus allows scripting). It is expected that this happens if we return the JSON with Content-Type `text/html`, but not with any of the others. Well maybe an empty Content-Type could be excused, but if the browser says `application/json, the browser should definitely not treat the contents as HTML.

# DOM-based XSS
It's expected that DOM-based XSS will be more commons in apps reying heavily on JavaScript, than what has been seen in traditional apps. DOM-based XSS occurs because user input is unsafely handled in javascript running on the page. We will address a few examples on this type of XSS. Some of these attacks never reach the server side, and thus cannot be detected by server side code.

## Insecure writes

Insecure writes happens when user input is written to the DOM without first being sanitized. Data can come in through user input in the browser, or it can be loaded through JSON from the server (which would mean it was a stored DOM-based XSS attack). Insecure writes means we are either outputting the data directly in the DOM using `.innerHTML` or through unsafe jQuery functions like:

	$.after()
	$.append()
	$.appendTo()
	$.before()
	$.html()
	$.insertAfter()
	$.insertBefore()
	$.prepend()
	$.prependTo()
	$.replaceAll()
	$.replaceWith()
	$.unwrap()
	$.wrap()
	$.wrapAll()
	$.wrapInner()
	$.prepend()

## Insecure code execution

As mentioned in [A1 - Injection](http://open.bekk.no/owasp-top-10-for-javascript-a1-injection/) the use of insecure functions in JavaScript can cause insecure code execution. More specifically this means using functions that build JavaScript from strings:

	eval(string)
	new Function(string)
	setTimeout(string, time)
	setInterval(string, time)

In addition to these we find API specific functions like `$.globalEval(string). As we can easily imagine dynamically building code containing user input is probably not a good idea, and is very likely to lead to problems down the road. JSHint, which is a common tool for quality checks on JavaScript, agrees: eval is evil.

## Insecure use of APIs

Insecure use of jQuery can also lead to unexpected XSS. As mentioned in jQuery XSS something as simple as this can lead to XSS: `$(location.hash)`

This can be exploited with:

	http://example.com/some/page#<img src=insecure onerror=alert(1)>

The next one is a bit more subtle (courtesy of [An overview of DOM XSS](http://sec.omar.li/2012/05/overview-of-dom-xss.html)):

	hash = location.hash.substring(1);
	if (!$('a[name|="' + hash + '"]')[0]) {
	    // not important
	}

An input of `"] <img src=insecure onerror=alert(1)>` would actually work here.

While the examples above jQuery specific, it is very likely that there are similar problems in other frameworks.

## Insecure use of document.location

Allowing user input to be directly assigned to document.location`, can easily lead to XSS in the form of `javascript:` URLs. The following example is from twitter back in september 2010 (I _highly_ recommend you read the full blog post here: [Minded Security Blog: A Twitter DomXss, a wrong fix and something more](http://blog.mindedsecurity.com/2010/09/twitter-domxss-wrong-fix-and-something.html)): 

	(function(g){var a=location.href.split("#!")[1];if(a){g.location=g.HBR=a;}})(window);

The code above, extracts whatever comes after `#!` in the url, and asigns it to `window.location`. The intention was to allow a url of `http://twitter.com/#!/webtonull` to be redirected to `http://twitter.com/webtonull`. However consider the following url:

	http://twitter.com/#!javascript:alert(1)

## Persistent client side XSS

If the XSS vector is persisted in Web Storage, and later rendered in an insecure way, we call it a persistent client side XSS. As the name implies this attack will never reach the server, but still trigger every time the vulnerable page is loaded in the given browser.

# XSS when using templates
Templating from frameworks like Mustache.js and underscore.js etc. allow javascript frameworks to clearly seperate view from data. They normally provide a set of tags for allowing coding and data output. The set from underscore.js looks like:

`<% %>` - view logic (JavaScript code)
`<%= %>` - data output (no escaping)
`<%- %>` - data output (HTML escaping)

The HTML escaping escapes `&`, `<`, `>`, `"`, `'` and `/`. I don't intend to pick on underscore.js. This is one of the better escaping functions I've seen. But we need to remember to use the HTML escaping tag. And there are contexts where this will not work, simply because HTML escaping is not the correct escaping.

**Quoteless HTML attributes**

Template code:

	<img title=<%- title %> ...>

Attack:

	<img title=something onclick=alert(1) ...>

**HTML comments** - courtesy of http://html5sec.org/#133

Template code:

	<!-- <%- title %> -->

Attack:

	<!-- ` I'm now outside the comment in IE6-8 -->

**Inside script tags**, but you are not using those in a javascript app, right?

Template code:

	<script>
	var id = <%- id %>
	</script>

Attack:

	<script>
	var id = 1;alert(/XSS/.source);
	</script>

**Inside javascript event handlers**, but you are not using those in a javascript app, right?

Template code:

	<img onmouseover="showToolTip('<%- title %>')" ...>

Attack:

	<img onmouseover="showToolTip('&#039;);alert(&#039;XSS')" ...>

**Insecure use of URLs**

Template code:

	<a href="<%- url %>" ...>

Attack:

	<a href="javascript:alert(1)" ...>

The above list is not complete. There are a lot more weird contexts. See [http://html5sec.org/] and the OWASP cheat sheets listed below.

# Third-party javascript
When you are adding direct links to third-party javascript files, instead of putting the files locally on your server, you have to remember that you are now allowing XSS from the domain hosting that file. That's regardless of whether you are loading a javascript library or "just" using JSONP to load data. Consider a `<script>` tag like this:

	<script src="http://example.com/example.js"></script>

Now if example.com goes rougue or is hacked, example.js could start delivering malware to your users or silently stealing their credentials when they log in.

Another thing to consider any security vulnerabilities in downloaded libraries. When you are pulling in some third party script, you should also make sure you later stay up to date, and follow any announcements from the developers of that library.

# Mitigation
* **Default to secure methods** - use `$.text()` instead of `$.html()`
* **Default to escaping output** - use `<%- %>` instead of `<%= %>`
* **Check your templating framework's escaping code** - is it as thorough as underscore.js or does it escape fewer characters?
* **Prefer frameworks with secure defaults**
* **Set the correct Content-Types and use proper JSON escaping**
* **Beware of XSS contexts** - make sure you are not using the wrong/insufficient escaping for the contexts
* **Avoid sticking user data in insecure functions like `eval`**
* **Beware when using user data with APIs like jQuery** - test it
* **Use automated tests with XSS attacks** - test your code and templates using automated javascript tests and bad data
* **Read and learn the OWASP XSS cheat sheets by heart**
* **Use [jQuery encoder](https://github.com/chrisisbeef/jquery-encoder) for complex contexts**

# Resources

* [OWASP XSS Prevention Cheat Sheet](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet)
* [OWASP Abridged XSS Prevention Cheat Sheet](https://www.owasp.org/index.php/Abridged_XSS_Prevention_Cheat_Sheet)
* [OWASP DOM-Based XSS Prevention Cheat Sheet](https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet)
* [An overview of DOM XSS](http://sec.omar.li/2012/05/overview-of-dom-xss.html)
* [jQuery XSS](http://ma.la/jquery_xss/)
* [HTML5 Security Cheatsheet](http://html5sec.org/)
* [jQuery encoder](https://github.com/chrisisbeef/jquery-encoder)

# Missing anything?
Contact me on [twitter](https://twitter.com/webtonull) or add a comment below, and I'll be happy to add (with attribution).