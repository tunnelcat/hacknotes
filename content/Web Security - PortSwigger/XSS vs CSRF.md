# What's the difference? 
- XSS allows an attacker to execute arbitrary JS in a victim's browser
- CSRF allows an attacker to induce a victim to perform unintended actions (like changing their password/email)

Usually XSS is more serious than CSRF.
- CSRF usually applies to a certain subset of unprotected actions
- XSS usually allows an attacker to induce ANY action that the victim can perform

The ability to read the responses only applies to XSS
- CSRF is "one-way", so while some requests can be induced, no responses can be retrieved
- XSS is "two-way", so arbitrary requests can be issued, and responses can be read/exfiltrated to external domains.

# Can CSRF tokens prevent XSS attacks? 
Some XSS attacks can be prevented with effective use of CSRF tokens. Simple reflected XSS vulnerabilities falls under this category because CSRF tokens will prevent forged cross-site requests. 

However: 
- If a reflected XSS is NOT protected by CSRF and exists somewhere on a site, then it is still an exploitable XSS.
- If an exploitable XSS exists somewhere on a site, then an attacker can induce actions even if they are protected by CSRF tokens (script can request the valid token).
- CSRF tokens do NOT protect against stored XSS vulnerabilities.