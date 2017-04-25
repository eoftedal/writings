# Why JSONP is a bad idea

A friend asked me to recap why JSONP is a bad idea. This is my response.

## Why JSONP?

JSONP was invented before CORS was available in browsers, as a means to load data cross site. The JSONP endpoint would
support a callback parameter and the response would be wrapper in this callback:

HTML on domain-a.com:

````
<script>
function load(data){
  //do something with data
}
</script>
<script src="http://domain-b.com/data?callback=load">
````

**Response from domain-b.com:**

````
load({"all":"the data", ... })
````

So domain-a.com defines a function load, and then passes the name of this function as `callback` in the script tag 
pointing to domain-b.com. When the script from domain-b.com is loaded, it automatically invokes the script because 
the returned script invokes the callback with the data as parameter(s).

Because cross domain script tags are not blocked by the [Same Origin Policy](https://en.wikipedia.org/wiki/Same-origin_policy), this would simply work.

## Private data leakage
Because this seemed like a simple way to load data, people started using it also for private data. The requests resulting from the script tags would of course include the cookies, and thus if the user was logged into domain-b.com, then domain-b.com could deliver content to domain-a.com's script.

The problem is that domain-b.com doesn't know if it's the "trusted" domain-a.com, or some completely different site loading the data. So any site could declare a callback and the script tag and steal data. If I remember correctly Gmail was vulnerable to this many years. A random site could include a script tag with callback and steal the user's address book.

## Content spoofing and CSP bypass due to lax callback validation
If a site has an injection vulnerability, and attacker may be tempted to inject javascript. Some sites block external URLs either due to input validation or Content Security Policy (CSP). But if the callback parameter is not validated, the attacked can include his/her own content such as:
````
http://domain-b.com/data?callback=doSomethingEvil();//
````
which would result in:
````
doSomethingEvil();//({"all":"the data", ... })
````
which essentially comments out the actual data, and adds arbitrary javascript.

There could even be the possibility to inject other content such as images, flash films (see [Rosetta flash](https://miki.it/blog/2014/7/8/abusing-jsonp-with-rosetta-flash/)) or html (for inclusion in iframes).

## There is a reason why we have CORS
CORS has a much better security model, and allows restriction based on origins. Use that instead.


