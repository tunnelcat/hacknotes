# What is the HTTP Host Header? 
The Host header is a mandatory request header (HTTP/1.1). It specifies the domain name that the client wants to access. 

For example, when accessing `https://portswigger.net/web-security`: 
```
GET /web-security HTTP/1.1
Host: portswigger.net 
```

# What is the purpose of the HTTP Host header? 
Purpose: identify which back-end component the client wants to communicate with.
Nowadays, multiple websites and apps can be accessible at the same IP address depending on the scenario.

## Virtual hosting 
Although websites on a single server may have different domain names, they all share a common IP. Websites hosted like this are known as "virtual hosts"

## Routing traffic via an intermediary
When websites are hosted on distinct back-end servers but traffic between client and server goes through some intermediary like a reverse proxy, load balancer, or CDN. All the domain names still resolves to the intermediary component. 

## How does the HTTP Host header solve this? 
In these scenarios (the target URL resolves to one IP address), the Host header is used to specify the intended back-end target. 

# What is an HTTP Host Header attack? 
Exploit websites that handle the value of the Host header in unsafe ways. If the server trusts the header, or fails to validate/escape it, there may be ways to inject harmful payloads ("Host header injection") and pass arbitrary hostnames to the target app. 

Example: 
```
<a href="https://_SERVER['HOST']/support">Contact support</a> 
```
Because the Host header is controllable, this practice can lead to: 
- Web cache poisoning
- Business logic flaws
- Routing-based SSRF
- Server-side vulns like SQL injection 

# Exploiting HTTP Host Header vulns
## Supply an arbitrary Host header
Try supplying an arbitrary Host header. Sometimes you can still access the target website due to some fallback or default settings. If your target website is set as the default, you might be able to do something. If not, you may get `Invalid Host header` (likely with CDN-accessed websites) and other techniques are needed. 

## Check for flawed validation
Sometimes the Host header is improperly parsed. Some examples below. 
![fca3f4328d1896c663b645fde42fc512.png](../../_resources/fca3f4328d1896c663b645fde42fc512-1.png)

## Send ambiguous requests
Sometimes there are discrepancies between the code that validates the host and the code that does something vulnerable. There may be a way to trick systems to send a request that appears to have a different host.

Try injecting duplicate Host headers and see what happens. Precendence? 
![b2011eb68a1a58e3111934450ff3141e.png](../../_resources/b2011eb68a1a58e3111934450ff3141e-1.png)

Try supplying an absolute URL. 
![b435dddb1878bdc25b731d2ea45c5b16.png](../../_resources/b435dddb1878bdc25b731d2ea45c5b16-1.png)

Try adding line wrapping. 
![ad837ac745c8ef066b5143b57e108f4d.png](../../_resources/ad837ac745c8ef066b5143b57e108f4d-1.png)

etc...

## Inject host override headers
If you can't override the Host header using ambiguous requests, try injecting into host override headers like `X-Forwarded-For`.
This may allow you to redirect the URL.
![8b2c861b1e2b5b8c8337e6d2edcc6e08.png](../../_resources/8b2c861b1e2b5b8c8337e6d2edcc6e08-1.png)

- `X-Forwarded-For`
- `X-Host`
- `X-Forwarded-Server`
- `X-HTTP-Host-Override`
- `Forwarded`

## Exploitation
### Password reset poisoning
1. Attacker resets user's password on their behalf, changing the Host header to a domain they control. 
2. The email is sent to the user, and if the URL depends on the Host header, the URL points to the attacker's domain. 
3. If the victim clicks the link, the password reset token can be exfiltrated. 
4. The attacker can supply the real token to the website and change the password. 

### Web cache poisoning via the Host header
If the Host header is reflected in the response, it may still not be exploitable because there is no way to force a victim's browser to issue an incorrect host in a useful manner. However, this changes when the target uses a web cache. There is a possibility that the poisoned response can be served to others. 

### SQL injection
Remember that every HTTP header is a potential vector for server-side vulnerabilities. The Host header may be susceptible to SQL injection. Try some payloads. 

### Classic Auth bypass
Sometimes access control makes flawed assumptions based on the Host header, allowing you to access restricted functionality. Try setting it to `localhost` or something.

### Accessing internal websites with virtual host brute-force
When publicly accessible websites and private, internal sites are on the same server, an attacker can access virtual hosts on servers they have access to (guessing/bruteforcing the hostnames/subdomains)

### Routing-based SSRF 
Exploits intermediary components like load balancers and reverse proxies. They basically receive requests and forward them to some back-end component. If this can be attacker-controlled, SSRF attacks can be performed on the internal network. You can use out-of-band methods (Burp Collaborator) to see if the DNS lookups from the target occur. This would mean that the component may be routing requests to arbitrary domains. The next step would be to scan (brute force?) possible private IPs on the internal network (like the standard `192.168.0.0/16` range)

# How to prevent HTTP Host Header attacks
- Avoid using the Host header in server-side code
- Double-check if you need absolute URLs. If not, use relative ones. 
- Validate/protect absolute URLs (not based on Host header)
- Validate the Host header using a whitelist/framework
- Don't support Host override headers like `X-Forwarded-Host`
- Forward requests to a whitelist on intermediary components (load balancers, reverse proxies)
- Avoid hosting internal-only websites and public-facing apps on the same server