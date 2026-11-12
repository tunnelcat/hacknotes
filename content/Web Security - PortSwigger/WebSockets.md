# WebSockets
- Initiated over HTTP 
- Provide long-lived connections with asynchronous communication
- Used for all kinds of purposes, including performing user actions and transmitting sensitive information
- WebSockets vulnerabilities are often analogous to the various HTTP vulnerabilities

# Manipulating WebSocket traffic
You can use Burp Suite (WebSockets history tab in Proxy and Repeater) to: 
- Intercept and modify WebSocket messages.
- Replay and generate new WebSocket messages.
- Manipulate WebSocket connections.

# WebSockets vulnerabilities
In principle, practically any web security vulnerability might arise in relation to WebSockets.

## Manipulating WebSocket messages to exploit vulnerabilities
Most input-based vulnerabilities can be found by tampering with the contents of WebSocket messages. For example, instead of `{"message":"Hello Carlos"}`, try injecting `{"message":"<img src=1 onerror='alert(1)'>"}` for a potential XSS, etc.

## Manipulating the WebSocket handshake to exploit vulnerabilities
Some WebSockets vulnerabilities can only be found and exploited by manipulating the WebSocket handshake. These vulnerabilities tend to involve design flaws, such as: 

- Misplaced trust in HTTP headers to perform security decisions, such as the X-Forwarded-For header.
- Flaws in session handling mechanisms, since the session context in which WebSocket messages are processed is generally determined by the session context of the handshake message.
- Attack surface introduced by custom HTTP headers used by the application.

This is sometimes necessary because: 
- It can enable you to reach more attack surface.
- Some attacks might cause your connection to drop so you need to establish a new one.
- Tokens or other data in the original handshake request might be stale and need updating.

 You can manipulate the WebSocket handshake using Burp Repeater:

- Send a WebSocket message to Burp Repeater.
- In Burp Repeater, **click on the pencil icon** next to the WebSocket URL. This opens a wizard that lets you attach to an existing connected WebSocket, clone a connected WebSocket, or reconnect to a disconnected WebSocket.
- If you choose to clone a connected WebSocket or reconnect to a disconnected WebSocket, then the wizard will show full details of the WebSocket handshake request, which you can edit as required before the handshake is performed.
- When you click "Connect", Burp will attempt to carry out the configured handshake and display the result. If a new WebSocket connection was successfully established, you can then use this to send new messages in Burp Repeater.

## Using cross-site WebSockets to exploit vulnerabilities
- Exploiting a CSRF misconfiguration/vuln on a WebSocket handshake leads to cross-site WebSocket hijacking attacks.
- This allows an attacker to make a cross-domain WebSocket connection from a web site that the attacker controls.
- Often has a serious impact, performing privileged actions or capturing data from a user.

 An attacker can create a malicious web page on their own domain which establishes a cross-site WebSocket connection to the vulnerable application. The application will handle the connection in the context of the victim user's session with the application.

The attacker's page can then send arbitrary messages to the server via the connection and read the contents of messages that are received back from the server. This means that, unlike regular CSRF, the attacker gains two-way interaction with the compromised application. 

In terms of the normal conditions for CSRF attacks, you typically need to find a handshake message that relies solely on HTTP cookies for session handling and doesn't employ any tokens or other unpredictable values in request parameters. 

For example, the following WebSocket handshake request is probably vulnerable to CSRF, because the only session token is transmitted in a cookie: 
```
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket 
```
> Note: The `Sec-WebSocket-Key` header contains a random value to prevent errors from caching proxies and is not used for authentication or session handling purposes. 


# How to secure a WebSocket connection
- Use the `wss://` protocol (WebSockets over TLS).
- Hard code the URL of the WebSockets endpoint, and certainly don't incorporate user-controllable data into this URL
- Protect the WebSocket handshake message against CSRF, to avoid cross-site WebSockets hijacking vulnerabilities.
- Treat data received via the WebSocket as untrusted in both directions. Handle data safely on both the server and client ends, to prevent input-based vulnerabilities such as SQL injection and cross-site scripting.