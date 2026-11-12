## What is access control? 
- Authentication - identifies user and confirms validity 
- Session management - identifies which HTTP requests are being made by that user
- Access control - determines if user is allowed to carry out some action

## Access control security models 
- Programmatic - matrix of user privileges stored in a database and access controls are applied accordingly
- Discretionary (DAC) - access based on users/groups with owners of resources able to delegate permissions
- Mandatory (MAC) - centrally controlled, access to objects constrained with some absolute standard, users/owners have no right to delegate permissions (Ex: military clearance levels)
- Role-based (RBAC) - named roles (or group names) are defined, then users are assigned to one or more roles for access. Pretty flexible and not bad if # of roles kept minimal

## Types of access controls
### Vertical access controls
- Example: admin user vs. normal user 
- Designed to enforce least privilege

### Horizontal access controls
- Example: users can only view their own account profiles
- Restricts access to specific user resources

### Context-dependent access controls
- Example: users cannot modify their cart after payment
- Restricts access to functionality/resources depending on the state of the app
- Prevents users from doing things in "wrong" order

## Unprotected admin functionality
- Sometimes, `/admin` panel is not protected at all
- But might also be concealed (less predictable URL) AKA security by obscurity...

![[3c95b29e1cf9f9ffd96a54e27eb7f57a-1.png]]

## Parameter-based access control
- Some app has user-controllable parameter or cookie that can easily be modified to get higher privileges

![[56c6e046266a00e806bb427dfc21a5c7-1.png]]

![[c711337fff1731537f627da15b061092-1.png]]

- May abuse certain functionality to get user-controlled parameters

![[61f114d89c5a3e63d068312ca27f57d0-1.png]]

![[b423d77d28a3ff8306a1683a161b10c5-1.png]]

![[4cc2a0b5f5342dab0ae2801c25504456-1.png]]

## Platform misconfigurations
- Sometimes, URL-based access controls are used (like denying POST request to certain URLs for certain roles), but they can be circumvented using non-standard HTTP headers like `X-Original-URL` and `X-Rewrite-URL` that overwrite the URLs

![[009114c5ce67c59bddb857f88a9c0d87-1.png]]

- Sometimes, web sites allow alternate HTTP request methods when performing actions. If an attacker can use `GET` or other methods to perform actions on a restricted URL, access control circumvention is possible.

Example conversion from `POST` to `GET` (use the "Change request method" button):

![[78d9c647450cbc232d309cc82387a2c7-1.png]]

![[9376fdd679b7021bc1153f30aa0bf0c6-1.png]]

## User ID controlled by request parameter
- Changing the User ID may be trivial in the URL parameters

![[dd5b014901e6992a008f30217d894b26-1.png]]

- The User ID might be unpredictable, but if it can be found somewhere, might not be a problem

- Changing the User ID might redirect you (because "invalid" login), but if there is data leakage in the body of the redirect, might have useful info disclosure

## Horizontal to vertical privilege escalation
- Sometimes, horizontal access can lead to vertical privesc if the compromised user is an admin, for example

## Insecure direct object reference (IDOR)
- If objects are sequenced (ex. numbers) or predictable, and user-supplied input allows access to different objects, entering a range of (numbers, indexes, etc) can reveal "hidden" objects

![[f8d70ba3eb6d59e10697a9b1965c0174-1.png]]

## Vulnerabilities in multi-step processes
- In some workflows, the first steps are properly controlled but not the last...so just skip the first 

## Referer-based access control
- Referer header indicates the page from which a request was initiated
- Spoofing the Referer header can trick the app to think that you have access (since the workflow requires authentication or something)

![[43950dc7e991f2a07ee45f92cbb77b80-1.png]]

