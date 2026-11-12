Basic structure is `header.payload.signature`, each encoded in base64url, a base64 variant where there is no padding (the `=`'s at the end) and where some non-alphanumeric characters are replaced with alternatives.

Example: 
`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6InVzZXIiLCJpYXQiOjE2NDE4NzQ3MzF9.xmiPYYD7nDPxqK1vsU-zNfYmsPOYd5FlRw1H9K82_D0`

```
{
  "alg": "HS256",
  "typ": "JWT"
}

{
  "username": "user",
  "iat": 1641874731
}

Signature = "xmiPYYD7nDPxqK1vsU-zNfYmsPOYd5FlRw1H9K82_D0"
```


Can use the JSON Web Tokens Burp extension, to attack tokens. Just send a request with JWTs to Repeater, then click Select extension... (need another extension called Hackvertor to have that option) on the top right of the pane and select the JSON Web Tokens extension.
