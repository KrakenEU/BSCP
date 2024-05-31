```
email registration has maximun length to 255 for example, it truncates the rest, you can register a company user:
-> very-long-string@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net (Make sure that the very-long-string is the right number of characters so that the "m" at the end of @dontwannacry.com is character 255 exactly.)
-> echo 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa' | wc
-> echo 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@dontwannacry.com.exploit-0aff00310328b657827b500001bb00d0.exploit-server.net' | wc


Current password to change it can be removed to access other account


While login, if you change the GET /role-selector request to /admin in intercept, your are administrator


Oracle encryption bypass (https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-authentication-bypass-via-encryption-oracle)
-> Stay logged in encoded cookie
-> Post comment with invalid mail ('invalid') sets notification cookie that is encrypted and decrypted in the next request with oracle encryption
-> If you copy your stay-logged-in cookie into notification cookie in decrypt tab you see wiener:1711060443422, so we know the cookie is user+timestamp
-> administrator:1711060443422 has appended "Invalid email address: " so in decoder we errase 23 bytes corresponding to that
-> we get an error saying we need to have a multiple of 16, so we padd it with 9 bytes to: xxxxxxxxxadministrator:1711060443422 and delete 32 bytes in encoder
-> Once you have the notification as administrator:timestamp correctly, copy it to stay-logged-in and delete the rest, access /admin
```
