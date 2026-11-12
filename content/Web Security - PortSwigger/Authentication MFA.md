## Simple bypass
- Sometimes, the 2FA is fake and you are already in the "logged in" state after you put in valid creds. This means that you can just browse to pages manually and just skip the 2FA altogether

## 2FA broken logic
- During initial login with valid creds, you may be assigned a (predictable) cookie to use in the 2FA stage. But, if you can swap the cookie with another user's, then you can login as an arbitrary user during the 2FA stage (might need to brute force the code)

Example: 
First, capture a legit user session cookie by logging in as a known user. Then, reuse the session cookie (changing the user parameter to the user you want access to, "carlos" in this case) and start brute-forcing the 2FA PIN with the same session (so the PIN doesn't change every time). 

![8bfae078a6ae58dc5ea7a0d6399a1cf7.png](../../_resources/8bfae078a6ae58dc5ea7a0d6399a1cf7-1.png)