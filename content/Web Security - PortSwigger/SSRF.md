## What is SSRF? 
- Server-side request forgery 
- Allows attacker to induce server-side app to make HTTP requests to some arbitrary domains
- Domains may be internal (access to internal systems)
- ...Or attacker-controlled external (may leak data)

## SSRF attacks against server
- Normal access controls can be bypassed when the server is induced to refer to itself

![[d33dfa6ea1f6679fb75d5242bcf39635-1.png]]

![[ef6687cef12d573909337272dfa1be5f-1.png]]

![[e728eb83ad05f69335ae36292daaf6b6-1.png]]

## SSRF attacks against other backend systems
- Can scan with Intruder payloads for backend systems

![[dc154bce0955ad85a2ed2c6b60f607df-1.png]]

## Circumvent blacklist input filters
- Many apps blacklist certain inputs like `127.0.0.1` or `localhost` or URLs like `/admin`
- Use alternatives for `127.0.0.1` like `2130706433`, `017700000001`, or `127.1`
- Register a domain name to resolve to `127.0.0.1` like `spoofed.burpcollaborator.net`
- Obfuscate blocked strings with URL encoding or case variation

## Whitelist input filters
- Much harder to circumvent (because whitelists are more restrictive)
- Try all sorts of things like: 
	- URL encoding
	- Double encoding
	- Other non-standard URL encodings
	- `evil-host#intended-host` (URL fragment)
	- `user@evil-host` (embedding credentials, or URLs...)
	- `intended-host.evil-host` (DNS hierarchy)
	- some combination like `evil-host#@intended-host/evil-path`

Example: 
![[f1817d4bbcab35cc1ecb89d17fd503f1-1.png]]

Explanation: `%2523` is double encoding for the `#` character, and it effectively "ignores" the `stock.weliketoshop.net` part of the URL. The `@` character fools the app into thinking that the real hostname is `stock.weliketoshop.net` when in reality, the URL is effectively `localhost:80/admin/delete?username=carlos`

## Bypass with open redirection
- If there is an open redirect somewhere, find it and use for SSRF bypass
- It works because the URL is first validated to be in the "trusted" domain, but requesting the URL will redirect to an arbitrary domain...

![[dbf8c270cac977df51b22d47009e3904-1.png]]

## What is Blind SSRF? 
- App can be induced to do a back-end HTTP request, but the response is not shown in the front-end response
- Impact is lower than full SSRF (one-way nature), but RCE is possible

## Blind with out-of-band detect
- Sometimes, analytics will fetch URLs located in headers (or somewhere else)

![[c7b13db93683da2f2b411fa6ee40a2f9-1.png]]

## TODO: Blind with Shellshock

