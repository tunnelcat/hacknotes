## Important note for WSA: XSS Exploit Server
- Since XSS can vary from browser to browser, WSA provides an exploit server that simulates sending an HTTP response to a victim user. 
- If the XSS doesn't trigger in your browser but you know it should, just send the lab URL (including any XSS payloads) that "should work" wrapped in an `<iframe src="url"/>` object and deliver the exploit to the victim. 
- There are also other ways to wrap the payload, such as with `<script>` etc.

Example ("Reflected XSS into HTML context with all tags blocked except custom ones"): 

![[d28d4bcef7c6cdc95e4745344e1798f0-1.png]]

## What is XSS? 
- Compromise interactions that users have with an app 
- Circumvents same origin policy (designed to segregate different websites from each other)
- Allow attacker to masquerade as victim user (and do stuff)
- Works by changing a website such that it returns bad JS to users (once it executes in victim's browser, compromised)

## XSS Proof of Concept
- Common practice to use `alert()` to demonstrate 
- However, `print()` is recommended (b/c of hitches with Chrome)

## Types of XSS
- Reflected - malicious script comes from current HTTP request
- Stored - malicious script comes from website's database
- DOM-based - vuln is in client-side code (not server-side)

# Reflected XSS
- Simplest
- Happens when app receives data in HTTP, then includes the data in the response insecurely
- Usually allows attacker to do anything the compromised user can...
- How to find? 
	- Test every entry point (parameters in URL and message, file paths, HTTP headers)
	- Submit random alphanumeric values (~8 chars) and see if reflected in response
	- Reflection context: text b/w HTML tags? in a JS string? etc.
	- Test a candidate payload (leave original random value and grep for it)
	- Test alternative payloads
	- Test the attack in a browser (copy-paste or Burp Intercept)

![[38595fa2f0a75d982d11668c93ccf41d-1.png]]

![[3a814837a5a4bc8e4e376a964a76b87c-1.png]]

![[b6a972c65ee8e2d14d0522fe317e495d-1.png]]

# Stored XSS
- AKA 2nd order/persistent XSS
- Happens when an app stores some (malicious) data, and shows the data with later HTTP responses in an unsafe way
- Example: posting malicious comments with `<script>` tags, then others view it
- Difference from reflected? 
	- Still can fully compromise user
	- Stored is self-contained in the app (no external way needed)
	- User more likely to be logged in at the time of triggered exploit 
- How to find? 
	- Need to identify entry/exit points and links b/w them, then test
	- Entries: parameters, query strings, URL path, HTTP headers, out-of-band (3rd party feeds? etc.)
	- Exits: any kind of HTTP responses returned to user...
	- This is not easy... but with time and persistence...

# DOM XSS
- Happens when website contains JS that takes attacker-controlled value (AKA "source") and passes to dangerous function (AKA "sink")
- Source: JS property like `location.search` that accepts data and attacker-controlled. Can also use `document.referrer`, `document.cookie`, etc.
- Sink: dangerous JS function/DOM object like `eval()` function in JS, or HTML sink like `document.body.innerHTML` (allows arbitrary injection)
- How to find? 
	- for HTML sinks, use CTRL+F with dev tools to see where the string is placed
	- for JS sinks, a little harder, use CTRL+SHIFT+F in Chrome to find what references the **source** with dev tools JS debugger and set breakpoints to see where the JS codepath ends up. Might be a useful sink...

# Exploiting XSS
- There are ways to steal cookies (user sessions), but it is limited: 
	- Victim might not be logged in 
	- Cookies hidden from JS using HttpOnly flag
	- Sessions may be locked using additional factors (IP addr)
	- Session might time out

