```
Change Host Header to your exploit server to exfiltrate password reset token

Bypass authorization by setting Host: localhost

Second Host Header reflected in response -> cache poison

SSRF to access admin panel somewhere in 192.168.1.0/24
-> confirm you can change host to your collaborator
-> deselect in intruder update host header to match target
-> Host: 192.168.0.§1§

You can add full URL as GET https://lab/ and then set arbitrary Host header
-> you can exploit again SSRF to admin in 192.168.1.0/24

Duplicate request with GET /admin and Host 192.168.0.1, in the first one restore it to normal GET / Host: lab, connection=keep-alive and send in sequence (in the exam it would be probably localhost:6566)
```
