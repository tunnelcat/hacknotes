# Takeaways
- After enum, turn over EVERY single rock. You never know if there's a better exploit waiting for you. Always look for the easiest way in. Don't complicate things that are simple. 
- Don't fall into rabbit holes. If a hole goes too deep, back out a little, scout for other holes, and hopefully you'll find the treasure in a reasonably-sized hole.

# Process
- First, do the standard lazy nmap scan...

```txt
# Nmap 7.91 scan initiated Mon Mar 29 14:27:24 2021 as: nmap -v -sS -A -Pn -T5 -p- -oN nmap.txt 10.10.10.3
Nmap scan report for 10.10.10.3
Host is up (0.052s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.6
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: OpenWrt White Russian 0.9 (Linux 2.4.30) (92%), Linux 2.6.23 (92%), Belkin N300 WAP (Linux 2.6.30) (92%), Control4 HC-300 home controller (92%), D-Link DAP-1522 WAP, or Xerox WorkCentre Pro 245 or 6556 printer (92%), Dell Integrated Remote Access Controller (iDRAC6) (92%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (92%), Linux 2.4.21 - 2.4.31 (likely embedded) (92%), Linux 2.4.27 (92%), Citrix XenServer 5.5 (Linux 2.6.18) (92%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 497.103 days (since Mon Nov 18 11:01:21 2019)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=203 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h04m54s, deviation: 2h49m45s, median: 4m52s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2021-03-29T14:33:41-04:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 445/tcp)
HOP RTT      ADDRESS
1   55.48 ms 10.10.14.1
2   55.45 ms 10.10.10.3

Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Mar 29 14:29:25 2021 -- 1 IP address (1 host up) scanned in 120.85 seconds
```

- Anonymous login for FTP is interesting... Also seems to have a samba server. Also, distccd v1 seems to have a public exploit with RCE? 

![[64beb32c7dee5f298d17cad61e07b607-1.png]]

![[453d957dd81ebd0dbb719b5a266e02b9-1.png]]

![[cec02b8be19b4ebc2376de4d7c4ab895-1.png]]

![[885b838f4a9eb8c44a029fed6fe5f9b1-1.png]]


- If we go this route, it seems that we need privilege escalation, but not sure what to use... 

```
searchsploit privilege escalation | grep -i linux | grep -i kernel | grep -i 2.6 | grep -i ubuntu | grep -i 8
```

- Maybe this isn't the way. Let's go back to the enumeration results and see if there is a faster way in for root privileges.
- Remember the Samba 3.0.20-Debian from nmap? Turns out there is a critical exploit.

![[474bde77c8ba6b68a2cc469a0e2699d1-1.png]]

- So we search for it in Metasploit and try to run it on the target host...

![[531378673a899bb84e19f06065fda46f-1.png]]

![[20b478b7a498f5c2d3240ef32279d499-1.png]]

- Yay! root is pwned. Now it's just as simple as a search for the flag files recursively.

![[5dc197bbb5e7696b52c034844197d388-1.png]]

- Done!

