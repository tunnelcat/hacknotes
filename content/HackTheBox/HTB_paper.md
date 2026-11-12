# Lessons learned
- DONT JUST RUN SCANS LOOK CLOSELY AT RESULTS...
- also on web examine any response headers you get with burp

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

ports 
- 22
- 80
- 443

apache 2.4.37, centos. openssh 8.0
![](Pasted%20image%2020231019155738.png)
![](Pasted%20image%2020231019155146.png)



## Initial Access

cgi shellshock? 
https://exploit-notes.hdks.org/exploit/web/cgi-pentesting/

find php files in root?  tried...


![](Pasted%20image%2020231019170947.png)

BRUH NOW I SEE A VIRTUAL HOST `office.paper`
![](Pasted%20image%2020231019171338.png)
![](Pasted%20image%2020231019171552.png)

on port 80, web server, wordpress?
![](Pasted%20image%2020231019171712.png)

wpscan results, WP version 5.2.3
![](Pasted%20image%2020231019173257.png)

secret content in drafts?
![](Pasted%20image%2020231019173415.png)

unauthenticated view private/draft posts
![](Pasted%20image%2020231019173524.png)
![](Pasted%20image%2020231019173629.png)

using the search function
![](Pasted%20image%2020231019173820.png)

more secrets?
![](Pasted%20image%2020231019174142.png)
![](Pasted%20image%2020231019174153.png)


Visiting `office.paper/?static=1`
![](Pasted%20image%2020231019174348.png)

```
test

Micheal please remove the secret from drafts for gods sake!

Hello employees of Blunder Tiffin,

Due to the orders from higher officials, every employee who were added to this blog is removed and they are migrated to our new chat system.

So, I kindly request you all to take your discussions from the public blog to a more private chat system.

-Nick

# Warning for Michael

Michael, you have to stop putting secrets in the drafts. It is a huge security issue and you have to stop doing it. -Nick

Threat Level Midnight

A MOTION PICTURE SCREENPLAY,
WRITTEN AND DIRECTED BY
MICHAEL SCOTT

[INT:DAY]

Inside the FBI, Agent Michael Scarn sits with his feet up on his desk. His robotic butler Dwigt….

# Secret Registration URL of new Employee chat system

http://chat.office.paper/register/8qozr226AhkCHZdyY

# I am keeping this draft unpublished, as unpublished drafts cannot be accessed by outsiders. I am not that ignorant, Nick.

# Also, stop looking at my drafts. Jeez!
```

rocketchat found!
![](Pasted%20image%2020231019174612.png)
register `test:test`
![](Pasted%20image%2020231019174632.png)

`recycleops` LFI?
![](Pasted%20image%2020231019175848.png)
![](Pasted%20image%2020231019175923.png)
![](Pasted%20image%2020231019180007.png)

LFI poc
![](Pasted%20image%2020231019180143.png)
![](Pasted%20image%2020231019180326.png)

![](Pasted%20image%2020231019180603.png)
![](Pasted%20image%2020231019180628.png)

rocketchat_password: `recycleops:Queenofblad3s!23`
![](Pasted%20image%2020231019180921.png)

reused creds to get into ssh `dwight:Queenofblad3s!23`

Queenofblad3s!23
![](Pasted%20image%2020231019181248.png)
dwight user pwned!

## Privesc
maybe useful? 
![](Pasted%20image%2020231019160533.png)

no sudo allowed.
![](Pasted%20image%2020231019181520.png)

writable PATH?
sudo 1.8.29
![](Pasted%20image%2020231019182332.png)
![](Pasted%20image%2020231019182442.png)

rocketchat creds `rocket:my$ecretPass`
my$ecretPass
![](Pasted%20image%2020231019182606.png)

Centos 8, kernel 4.18
![](Pasted%20image%2020231019191328.png)


mongodb stuff
![](Pasted%20image%2020231019191933.png)


I GOT STUCK HERE. bruh. why..............
it turns out linpeas didn't see a PolKit CVE-2021-3560 for some reason. 
- use other scripts? 
- update linpeas?

![](Pasted%20image%2020231019213803.png)
![](Pasted%20image%2020231019213818.png)

still no CVE-2021-3560...

Oh well. just run it as ippsec said :)
login to `secnigma:secnigmaftw`
and root pwned!
![](Pasted%20image%2020231019220712.png)

## Loot
user.txt
![](Pasted%20image%2020231019181306.png)
```
44b47cd8b0ace0a5699b004ad0ce1855
```


root.txt
![](Pasted%20image%2020231019220821.png)
```
db4895621a1232d8bb2993686a3a5986
```