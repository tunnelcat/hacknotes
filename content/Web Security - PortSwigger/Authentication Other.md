## Password reset broken logic
- Sometimes, the password reset function can be initiated for a known user account (with a known email account), but the actual password change form does not validate the token generated (effectively a useless security measure). This can enable arbitrary user password reset via submitting crafted HTTP requests to the form


## Brute-forcing a stay-logged-in cookie
- Sometimes the cookies for "remember me" can be predictably constructed. 

Example: 
`d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw` can first be decoded using base64 to get `wiener:51dc30ddc473d43a6011e9ebba6ca770` then using md5 on the second part, we get (the password) `peter`

- In this fashion, the password can effectively be brute-forced (by going through the process in reverse with a Python-generated wordlist) without dealing with lockouts and timers

![[52b2d9992800e54c88bab06145f3dd3e-1.png]]

- So, since the payload was `Y2FybG9zOmQ4NTc4ZWRmODQ1OGNlMDZmYmM1YmI3NmE1OGM1Y2E0`, the decoded credentials are `carlos:qwerty`

## Brute-force via password change
- The password change form may allow changes to any arbitrary user via hidden inputs. If so, there may be a brute-force vulnerability (remember to check errors on fail cases like "current password is incorrect" or "the new passwords do not match")

![[097b239237e1b0bbed3b686f8211ad94-1.png]]