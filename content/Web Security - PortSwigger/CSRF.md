# What is CSRF? 
Cross-site request forgery allows an attacker to induce users to perform actions they did not intend. It partially circumvents the same origin policy, which is designed to prevent different websites from interfering with each other.

# Impact
An attacker can change the email address or password of an account. If the compromised user is privileged, the attacker gains full control of the app.

# How does CSRF work? 
Three key conditions: 
- Relevant action: like changing passwords, modifying privileges
- Cookie-based session handling: Application relies solely on session cookies to ID
- No unpredictable request parameters: The requests to perform an action don't contain any parameters that the attacker doesn't know or can't guess. (Ex: when a password change function requires the existing password unknown to the attacker, no vulnerability is present)

An attacker can then construct a web page with some HTML similar to: 
![[6c948d76e2f33d6f9684c8ce5f53b694-1.png]]

When the victim visits the site, the browser will use the victim's cookie to perform an email address change. The `<script>` included submits the form automatically without user interaction.

# How to construct a CSRF attack
Manually creating the HTML needed for CSRF can be a pain, but Burp Pro provides a CSRF PoC generator. There are also free generators online like https://security.love/CSRF-PoC-Genorator/ [sic]

# How to deliver a CSRF exploit
Delivery mechanisms for CSRF are basically the same as reflected XSS. The attack can be placed as HTML into an attacker-controlled website, malicious links, or even stored as malicious user comments on a website.

Some simple CSRF exploits use the GET method and can be fully self-contained in a single URL on a vulnerable website. In this case, no external site is needed, and the attack would look like: 
![[c961fe9efe80d10abeec06a12661236c-1.png]]

# Preventing CSRF attacks
The most robust way is to include a CSRF token with relevant requests. The token should be: 
- Unpredictable with high entropy
- Tied to user session (not just a global pool of tokens)
- Strictly validated before relevant actions are executed

# Common CSRF vulnerabilities
Many interesting CSRF vulnerabilities happen because of mistakes in the validation of CSRF tokens.

## Validation of CSRF token depends on request method
Some apps correctly validate the token when the POST request is used, but not when another request like GET is used. Simply use an alternate request method in your payload to bypass validation.

## Validation of CSRF token depends on token being present
Some apps validate the token when it is present but skip validation if it is not. In this case, remove the csrf parameter entirely and deliver the attack.

## CSRF token is not tied to the user session
Some apps don't validate that a token belongs to the same session as the user making the request. So, the app accepts any token that appears in the global pool of tokens. An attacker can login to their own account, get a token, and reuse it for the CSRF attack.

Note: try to capture the CSRF token without using it first (since it will be invalidated after reuse)

## CSRF token is tied to a non-session cookie
(Variation of previous)
Some apps tie the CSRF token to a cookie, but not to the same cookie that is used to track sessions. This happens often when an app uses different frameworks for session handling and CSRF protection (not integrated together).

Example: 
![[0f0f6d9eaf4cd8fe8f52e4112b7345fb-1.png]]

An attacker can login to their own account, get a valid token and associated cookie, set the cookie somehow, and reuse the CSRF token.

Setting the cookie is not so trivial, and it often requires some functionality that sets a Set-Cookie header on the website itself.

It may take a multi-step exploit to set the cookie and inject the CSRF token. One way of doing this is to use an `img` block to inject the cookie via a URL (if a cookie can be injected this way), then submit some form (for the action) in the `onerror` event handler

```
 <img src="$cookie-injection-url" onerror="document.forms[0].submit()"> 
```

The above method works well when there is a cookie injection vulnerability (like HTTP CLRF injection using the `%0d%0a`): 

```
GET /?search=nothing%0d%0aSet-Cookie:+csrfKey=pwned HTTP/1.1
```

![[e033692fb4d22d910e21d0a1fb197458-1.png]]

## CSRF token is simply duplicated in a cookie
Going off the previous example that included a cookie injection, simply duplicating the CSRF token in a cookie does not work because if the token can be reused from the global pool, then so can the cookie. 

# Referer-based defenses against CSRF
Some apps use the `Referer` header (misspelled in the HTTP spec) to attempt defense against CSRF attacks by verifying that requests originated from the app's own domain.

## Validation of Referer depends on header being present
Some apps validate the `Referer` header, but skips the validation if the header is omitted. There are ways to drop the Referer header in the request (be wary of the SPELLING): 

```
<meta name="referrer" content="never">
```

## Validation of Referer can be circumvented
Some applications validate the `Referer` header in naive ways that can be bypassed. 

If the app checks the domain to start with an expected value, one bypass would be: 
```
http://vulnerable-website.com.attacker-website.com/csrf-attack 
```

If the app checks only for the inclusion of its domain name, one bypass would be: 
```
http://attacker-website.com/csrf-attack?vulnerable-website.com 
```

Note: many browsers strip the query string from the `Referer` header by default for security, but you can include the `Referrer-Policy: unsafe-url` header in your exploit response to override this.

There are also ways to spoof the `Referer` using some JavaScript to change the browser history like: 
```
history.pushState("", "", "/?your-lab-id.web-security-academy.net")
```