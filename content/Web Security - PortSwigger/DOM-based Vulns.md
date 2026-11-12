# What is the DOM? 
- Document Object Model
- Web browser's hierarchical representation of elements on the page.
- DOM-based vulns happen when a website has JS that takes an attacker-controlled value (a source) and passes it into a dangerous function (a sink).

# Taint-flow vulnerabilities
Many DOM-based vulns arise from the way client-side code manipulates attacker-controlled data. 

A **source** is a JavaScript property that accepts data that is potentially attacker-controlled. An example of a source is the `location.search` property because it reads input from the query string, which is relatively simple for an attacker to control. Ultimately, any property that can be controlled by the attacker is a potential source. This includes the referring URL (exposed by the `document.referrer` string), the user's cookies (exposed by the `document.cookie` string), and web messages. 

A **sink** is a potentially dangerous JavaScript function or DOM object that can cause undesirable effects if attacker-controlled data is passed to it. For example, the `eval()` function is a sink because it processes the argument that is passed to it as JavaScript. An example of an HTML sink is `document.body.innerHTML` because it potentially allows an attacker to inject malicious HTML and execute arbitrary JavaScript. 

The most common source is the URL, which is typically accessed with the `location` object. Consider the following vulnerable code: 

```
goto = location.hash.slice(1)
if (goto.startsWith('https:')) {
  location = goto;
} 
```

The code extracts the value of the `location.hash` property. So, an attacker could redirect users to `https://www.evil-user.net` with the following URL payload:

```
https://www.innocent-website.com/example#https://www.evil-user.net 
```

## Common sources
```
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
location
document.cookie
document.referrer
window.name
history.pushState
history.replaceState
localStorage
sessionStorage
IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB)
Database 
```

## Common sinks
![[e8738722928d4e9276998402ed11d856-1.png]]

# Web message vulns
## Basic web message attack
If a page handles web messages in an unsafe way (not verifying the origin, for example), properties and functions called by the event listener can become sinks. For example, an attacker could host a malicious `iframe` and use the `postMessage()` method to pass web message data to the vulnerable event listener, which then sends the payload to a sink.

Consider the following: 
```
<script>
window.addEventListener('message', function(e) {
  eval(e.data);
});
</script> 
```

This can be exploited by constructing the frame: 
```
<iframe src="https://vulnerable-website" onload="this.contentWindow.postMessage('print()','*')"> 
```

Notice that `postMessage()` specifies the `targetOrigin` as `'*'`, and this works because the event listener does not verify the origin of the message.

Consider the following: 
```
<script>
    window.addEventListener('message', function(e) {
        var url = e.data;
        if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) {
            location.href = url;
        }
    }, false); 
</script>            
```

The code has an open redirection vulnerability. This can be exploited by sending the following message to trigger XSS via JavaScript URL: 
```
javascript:print(1);//http:
```

## Origin verification
If an event listener includes some origin verification, the step can sometimes be flawed.

Consider the following: 
```
window.addEventListener('message', function(e) {
  if (e.origin.indexOf('normal-website.com') > -1) {
    eval(e.data);
  }
}); 
```

This code just checks if "normal-website.com" is anywhere in the URL, so the following can be injected: 
```
http://www.normal-website.com.evil.net
```

# DOM-based open redirect
Happens when a script writes attacker-controlled data to a sink that can trigger cross-domain navigation. For example, consider the following: 
```
let url = /https?:\/\/.+/.exec(location.hash);
if (url) {
  location = url[0];
} 
```
This code handles `location.hash` in an unsafe way. 

Open redirects can facilitate phishing attacks, or in some cases, when the start of the URL can be controlled, a Javascript injection with the `javascript:` pseudo-protocol could execute arbitrary code.

Example: 
```
<a href='#' onclick='returnURL' = /url=https?:\/\/.+)/.exec(location); if(returnUrl)location.href = returnUrl[1];else location.href = "/"'>Back to Blog</a>
```

This code handles the attacker-controlled  `url` parameter in an unsafe way, allowing for an open redirect using the following URL: 
```
https://vulnerable-site.net/post?postId=4&url=https://your-exploit-server.net/ 
```

# DOM-based cookie manipulation
DOM-based cookie manipulation happens when a script writes attacker-controlled data into a cookie value. An attacker may be able to use this to construct a URL (sent to the victim) that will set arbitrary values in select cookies. It may be a low-severity bug, but it has potential to be part of an exploit chain for a high-severity exploit. For example, consider the following: 
```
document.cookie = 'cookieName='+location.hash.slice(1); 
```

The impact of cookie manipulation depends on how the cookies are used. If the cookie is used for certain user actions, the attacker can perform any actions that the user can. 



# How to prevent DOM-based taint-flow vulnerabilities

# DOM clobbering
Skipped