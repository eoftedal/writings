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

Because cross domain script tags are not blocked by the Same Origin Policy, this would simply work.

## Private data leakage


## Content spoofing
