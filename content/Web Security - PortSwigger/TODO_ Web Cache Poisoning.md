# What is web cache poisoning? 
A technique that exploits a web server and cache behaviors so that a harmful HTTP response is served to other users.

Two phases: 
- Figure out how to elicit a response from the backend that has some dangerous payload. 
- Make sure the malicious response is cached and served to other victims.

# How does a web cache work? 
The cache is used to reduce latency and overload issues due to sending the same HTTP response to identical requests repeatedly. It sits between the server and user, where it caches responses to particular requests (usu. for some amount of time). If another user sends an equivalent request, the cached response is sent.

![[a6cd7d0b6faf44dc65c153d89deee884-1.png]]

## Cache keys
Caches identify equivalent requests by comparing a subset of the request's components (collectively known as the "cache key"). Typically, it can include the request line and `Host` header. All other components not included in the cache key are called "unkeyed" components. 

Most importantly, the cache ignores the unkeyed components, and this may enable malicious attacks. 

# Impact of web cache poisoning
The impact largely depends on how harmful the payloads the attacker can cache are. Also, how much traffic is on the affected page? 

# Constructing a web cache poisoning attack
Here are the basic steps: 
1. Identify and evaluate unkeyed inputs
2. Elicit a harmful response from the backend
3. Get the response cached

## 1. Identify (Manual/Param Miner)
Web cache poisoning attacks rely on the manipulation of unkeyed inputs, such as headers, to inject payloads. You can identify unkeyed inputs manually using Burp Comparer (comparing responses w/ and w/o the injected input) or use the Param Miner extension to automate this process!

Usage: 
Right-click on a request and select "Guess headers"
Logs will be in the "Issues" pane (Burp Pro) or "Extender -> Extensions -> Param Miner -> Output" tab of Param Miner (Burp Community)

**Warning**: On a live website, you don't want to poison the cache during testing on real users. So, make sure your requests have a unique cache key so that the response is served only to you ("cache buster"). An easy way to do this is to add a unique parameter to the request line like `/?cb=123`

## 2. Elicit harmful response
Evaluate how the website process the inputs to elicit a harmful response. For example, maybe a reflection can make reflected XSS possible.

## 3. Get response cached!
This step can be tricky. Whether a response is cached depends on file extension, content type, route, status code, response headers, etc. Play araound with requests on different pages and pay attention to the cache behavior.

# Exploiting cache design
## Web cache poisoning XSS
![[8534952cb0d638fa88a19149b84beee1-1.png]]

If the `X-Forwarded-Host` header in this example is unkeyed, then all users accessing `/en?region=uk` can be served an XSS payload: 

![[6094691fb8eb89b992b2908411e89a7e-1.png]]

## Unsafe resource imports
> Note: solved but bugged on WSA

If a dynamically generated URL is used for importing resources, an attacker may be able to manipulate a unkeyed header/input and point it to a malicious JS file instead.

![[b796686b2d8fc2795be783569c8d5ab2-1.png]]

Example attack: 

![[8aa892e091421aa289464c82458852c8-1.png]]

![[356f2b3a2e2e44b44a621b79f75b8339-1.png]]

## Unkeyed cookie 
> Note: solved but bugged on WSA

Cookies are often used to dynamically generate content. A common example is a language selector cookie. If the cookie is unkeyed, subsequent users may see a different language than they expect due to web cache poisoning.

![[be0cce770581481de8722bfb140f14a9-1.png]]

# Exploiting cache implementation


# How to prevent web cache poisoning