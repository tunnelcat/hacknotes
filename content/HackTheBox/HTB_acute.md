# Lessons learned
- BROWSE THE PAGES AND CLICK STUFF.
- seriously, it may look like the pages are all filler, static pages, but they hide some osint stuff in there sometimes, links to docs, pics that have metadata.
- if you see any suspicious folders, enumerate hidden files with `ls -hidden` 
	- for example, `c:\utils`
- if there's AV, maybe look around, try different folders to enumerate excluded dirs?

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

## Initial Access
viewing the https website for the cert
![](Pasted%20image%2020231020101736.png)
![](Pasted%20image%2020231020101633.png)

adding to /etc/hosts
![](Pasted%20image%2020231020101715.png)

website
![](Pasted%20image%2020231020101803.png)
wappalyzer
![](Pasted%20image%2020231020101828.png)

feroxbuster
![](Pasted%20image%2020231020113152.png)
![](Pasted%20image%2020231020113131.png)

wpscan attempt
![](Pasted%20image%2020231020102118.png)
wpscan attempt 2 (feroxbuster found /Acute directory)
![](Pasted%20image%2020231020113100.png)

iis shortname scanner
![](Pasted%20image%2020231020121518.png)

hmm, there's a doc?
```
ffuf -c -w ~/HTB/acute/custom_fuzz.txt -u https://atsserver.acute.local/FUZZ -e .doc,.docm,.docx
```
![](Pasted%20image%2020231020122257.png)

I'm SO DUMB HOLY SMOKES. GOT STUCK BUT.
CLICK ALL PAGES AND BROWSE THEM IDIOT!

new starter forms page on about.html
![](Pasted%20image%2020231020124030.png)

URLs
```
https://atsserver.acute.local/Staff
https://atsserver.acute.local/Staff/Induction
```
![](Pasted%20image%2020231020124150.png)

internal apps, `Password1!`
![](Pasted%20image%2020231020124422.png)

lois is the site admin
![](Pasted%20image%2020231020124531.png)

URL
```
https://atsserver.acute.local/Acute_Staff_Access
```
![](Pasted%20image%2020231020124550.png)

dc_manage restrictions?
![](Pasted%20image%2020231020130205.png)


powershell web access site
windows server 2016
![](Pasted%20image%2020231020124642.png)

exiftool reveals internal information

```
PC name: Acute-PC01
Daniel
FCastle

University of Marvel?
```
![](Pasted%20image%2020231020124902.png)

more users on the website!!!
```
aileen
charlotte
evan
ieuan
joshua
lois
```

extrapolated from the "FCastle" user found on the docx.
```
FCastle
AWallace
CHall
EDavies
IMonks
JMorgan
LHopkins
```
![](Pasted%20image%2020231020131539.png)

LOGGED IN with `EDavies:Password1!`
![](Pasted%20image%2020231020132434.png)
![](Pasted%20image%2020231020132234.png)

also `CHall:Password1!` seems to work, but no authorization on Acute-PC01
![](Pasted%20image%2020231020132409.png)

tree /F
![](Pasted%20image%2020231020132541.png)

myplaylist.wpl has Desktop.mp4? someone's taking video of the Desktop, etc. so we can screenshot??
![](Pasted%20image%2020231020165550.png)
but doesnt exist.
![](Pasted%20image%2020231020165709.png)

recycle bin?
![](Pasted%20image%2020231020165848.png)



winpeas
![](Pasted%20image%2020231020134026.png)
THERE'S AV???? UGH.
![](Pasted%20image%2020231020134221.png)

whoami /groups
![](Pasted%20image%2020231020134437.png)

Get-LocalUser
![](Pasted%20image%2020231020134506.png)

administrators
![](Pasted%20image%2020231020134536.png)

users directory
![](Pasted%20image%2020231020134946.png)

hoaxshell?

![](Pasted%20image%2020231020172646.png)
![](Pasted%20image%2020231020172658.png)

screenshots of screen
https://github.com/redcanaryco/atomic-red-team/blob/f339e7da7d05f6057fdfcdd3742bfcf365fee2a9/atomics/T1113/T1113.md#atomic-test-6---windows-screen-capture-copyfromscreen
![](Pasted%20image%2020231020173823.png)

transferring with hosted post.php and php dev webserver
```php
<?php 
$postdata = file_get_contents("php://input"); 
file_put_contents("outfile.txt", $postdata);
?>
```

![](Pasted%20image%2020231020174121.png)
![](Pasted%20image%2020231020174126.png)
FAIL to get screenshots.

found a folder that is excluded in windows defender
![](Pasted%20image%2020231020174916.png)

Running winpeas in the excluded directory
![](Pasted%20image%2020231020175046.png)
grab the out.txt with the php webserver upload method like last time.

meterpreter upgrade time.
![](Pasted%20image%2020231020180202.png)
![](Pasted%20image%2020231020180335.png)
![](Pasted%20image%2020231020180325.png)
getting screenshot is easy now. ( i made that weird recyclebin file on the desktop, so nothing there. )
![](Pasted%20image%2020231020180516.png)
![](Pasted%20image%2020231020180606.png)
WOOOAHHH SO COOL there's a simulated user!!!

![](Pasted%20image%2020231020211438.png)

![](Pasted%20image%2020231020211619.png)
![](Pasted%20image%2020231020211416.png)

Got Creds for ATSSERVER? possibly Acute-PC01 too.
`acute\imonks:W3_4R3_th3_f0rce.`
W3_4R3_th3_f0rce.

lateral movement?? this is crazy. so we need to log in to atsserver? maybe.
![](Pasted%20image%2020231020212839.png)

```
$password=ConvertTo-SecureString "w3_4R3_th3_f0rce." -AsPlainText -force
$creds2=New-Object System.Management.Automation.PSCredential("acute\imonks", $password)
Enter-PSSession -ComputerName atssever -Credential $creds2
```
doesn't work :(
![](Pasted%20image%2020231020213532.png)

but copying the screenshare, we can try doing the ConfigurationManager dc_manage?
![](Pasted%20image%2020231021000257.png)

bruh. the w was capital in the password. it's just hard to read from the screenshot.

getting all the commands available to us
```
Invoke-Command -ComputerName ATSSERVER -Credential $cred -ConfigurationName dc_manage -ScriptBlock { Get-Command } 
```
![](Pasted%20image%2020231021002125.png)
![](Pasted%20image%2020231021002443.png)




## Privesc



## Loot
user.txt
```

```


root.txt
```

```