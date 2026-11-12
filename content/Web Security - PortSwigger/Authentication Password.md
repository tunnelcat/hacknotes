## User enumeration

- Take a look around the website (if you have access) and try to guess what usernames
    
- In HTTP responses sometimes there are usernames
    
- Predictable patterns of usernames?
    
- Response anomalies
    - Different status codes
    - Error messages (Invalid user/taken, or sometimes invisible difference somewhere)
    - Consistently/considerably slower/faster response times

Example of response time testing with an intentionally long and incorrect password (consistently at the top when sorted): 

![[d4ded586192dc3c869283be479ef7918-1.png]]

![[cc683f63389268f3e7b00e21ef1d4779-1.png]]

![[636d431fdae674f32d44561952b6dc54-1.png]]

- Note: can use Burp Intruder to do simple brute-force attacks and see obvious anomalies
    
![[9d8c33159f261da8972ddbdcac1e0d0a-1.png]]

![[bcfb229686718fb6a15c654e4b48b63d-1.png]]

## Simple IP blocks
- Can be bypassed by adding HTTP headers (often, 127.0.0.1 is trusted by the webapp)
- Often, the `X-Forwarded-For` header is sufficient for bypassing simple IP blocks

```txt
X-Originating-IP: 127.0.0.1 
X-Forwarded-For: 127.0.0.1 
X-Remote-IP: 127.0.0.1 
X-Remote-Addr: 127.0.0.1
```


## Flawed brute-force protection

- Sometimes, flawed logic is used. For example, the fail attempt counter is reset every time a valid login is entered. (So just interleave valid credentials once in a while)
    
- Hint: Throttle the requests artificially with the Resource Pool tab (so that it doesn't do a million concurrent requests and lock you out)


![[615c3f1ed9b390969af13f2d524af07d-1.png]]

## Account locks

- Can abuse account locks to enumerate usernames (made too many attempts, meaning user is real)

![[4f40a16de3771bc55df93aafa39d3b1d-1.png]]

## Multiple credentials per request

- Sometimes, using objects for credentials leads to object "smuggling" in a single request

Example: clearly some sort of JSON object being passed through

![[e33b1a0dfbed25ba98812f6549da68f7-1.png]]

So "smuggling" a bunch of passwords with a JSON array (Google the syntax) of passwords...

![[2cbfe5cd962046e9f60c8ce2e19c3a90-1.png]]

And, eventually you can forward the whole request to the browser using Burp Proxy (after you catch a generic login HTTP request, copy-paste and forward this HTTP request instead)
NOTE: You can also right click on the request in Repeater and use the "Request in browser" option...