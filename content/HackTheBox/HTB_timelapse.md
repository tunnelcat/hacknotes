# Lessons learned

# Writeup 
## Recon

### rustscan

```
sudo rustscan -a <ip> | tee rustscan.txt
```

### nmapAutomator

```
sudo nmapAutomator.sh -H <ip> -t Recon 
```

## autorecon

```
sudo python3 autorecon.py -t targets.txt -v
```

DC01
![](Pasted%20image%2020231019222027.png)

## Initial Access
getting guest access
![](Pasted%20image%2020231019223732.png)

smbclient access, download recursively
![](Pasted%20image%2020231019224002.png)
![](Pasted%20image%2020231019224026.png)

password protected winrm zip
![](Pasted%20image%2020231019224133.png)

zip2john, then remove the beginning part to go into hashcat
![](Pasted%20image%2020231019224836.png)
![](Pasted%20image%2020231019224713.png)
![](Pasted%20image%2020231019224730.png)
Cracked!
PW: supremelegacy
![](Pasted%20image%2020231019224808.png)

unzipping with pw
![](Pasted%20image%2020231019224912.png)

but need to extract key and pem files from pfx to use with evil-winrm.
tried `supremelegacy` but didn't work.
![](Pasted%20image%2020231019225112.png)

cracked the pfx, time to get to work.
![](Pasted%20image%2020231019230130.png)
![](Pasted%20image%2020231019230110.png)

getting the key and cert, where I set the PEM passphrase to `password` 
then last step is just removing that password.
![](Pasted%20image%2020231019230404.png)
![](Pasted%20image%2020231019230409.png)
![](Pasted%20image%2020231019230559.png)

user legacyy pwned!
![](Pasted%20image%2020231019230747.png)



## Privesc
winpeas64.exe

whoami /priv
![](Pasted%20image%2020231020010306.png)

whoami /groups
![](Pasted%20image%2020231020010252.png)

users
![](Pasted%20image%2020231020010341.png)

PS history ? CREDS
User: svc_deploy
E3R$Q62^12p7PLlC%KWaxuaV
![](Pasted%20image%2020231020011035.png)
![](Pasted%20image%2020231020011425.png)

GetNPUsers
![](Pasted%20image%2020231020011625.png)

GetUserSPNs
![](Pasted%20image%2020231020011602.png)

GetADUsers
![](Pasted%20image%2020231020012705.png)

ldapdomaindump
![](Pasted%20image%2020231020013511.png)

password spraying
![](Pasted%20image%2020231020012902.png)
![](Pasted%20image%2020231020012732.png)
![](Pasted%20image%2020231020012935.png)



bloodhound-python
![](Pasted%20image%2020231020011922.png)
![](Pasted%20image%2020231020011933.png)

bloodhound
![](Pasted%20image%2020231020015529.png)
![](Pasted%20image%2020231020012110.png)
"find shortest paths to domain admins"
![](Pasted%20image%2020231020012552.png)

logging into svc_deploy
![](Pasted%20image%2020231020013120.png)

svc_deploy is in LAPS_Readers according to ldapdomaindump
![](Pasted%20image%2020231020013624.png)

use pyLAPS to extract LAPS passwords
DC01$ Administrator PWNED! creds:
9}p4m6D#.+m+($62J29hRs7O
![](Pasted%20image%2020231020014159.png)
![](Pasted%20image%2020231020014411.png)

Administrator pwned!
![](Pasted%20image%2020231020014556.png)
running `tree /F` in `c:\users`
![](Pasted%20image%2020231020014647.png)
![](Pasted%20image%2020231020014637.png)

## Loot
user.txt
![](Pasted%20image%2020231020010622.png)
```
9afba1017064184ee4bb3b59911da576
```


root.txt
![](Pasted%20image%2020231020014704.png)
```
4e39b2803ed4b87120ee9a347c84dbaf
```