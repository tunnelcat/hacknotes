# Process
- FIrst, a standard nmap scan 

```sh
sudo nmap -v -sS -A -Pn -T5 -p- -oN nmap.txt 10.10.10.40
```

```txt
└─$ 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-31 15:13 EDT
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 15:13
Completed NSE at 15:13, 0.00s elapsed
Initiating NSE at 15:13
Completed NSE at 15:13, 0.00s elapsed
Initiating NSE at 15:13
Completed NSE at 15:13, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 15:13
Completed Parallel DNS resolution of 1 host. at 15:13, 0.07s elapsed
Initiating SYN Stealth Scan at 15:13
Scanning 10.10.10.40 [65535 ports]
Discovered open port 445/tcp on 10.10.10.40
Discovered open port 135/tcp on 10.10.10.40
Discovered open port 139/tcp on 10.10.10.40
Increasing send delay for 10.10.10.40 from 0 to 5 due to 281 out of 702 dropped probes since last increase.
SYN Stealth Scan Timing: About 7.77% done; ETC: 15:19 (0:06:08 remaining)
SYN Stealth Scan Timing: About 14.96% done; ETC: 15:19 (0:05:47 remaining)
Discovered open port 49154/tcp on 10.10.10.40
Discovered open port 49153/tcp on 10.10.10.40
SYN Stealth Scan Timing: About 22.05% done; ETC: 15:20 (0:05:22 remaining)
SYN Stealth Scan Timing: About 29.14% done; ETC: 15:20 (0:04:54 remaining)
SYN Stealth Scan Timing: About 36.09% done; ETC: 15:20 (0:04:27 remaining)
SYN Stealth Scan Timing: About 43.05% done; ETC: 15:20 (0:03:59 remaining)                                                                                                                             
Discovered open port 49157/tcp on 10.10.10.40                                                                                                                                                          
SYN Stealth Scan Timing: About 50.23% done; ETC: 15:20 (0:03:29 remaining)
Discovered open port 49155/tcp on 10.10.10.40
SYN Stealth Scan Timing: About 56.90% done; ETC: 15:20 (0:03:03 remaining)
Discovered open port 49156/tcp on 10.10.10.40
SYN Stealth Scan Timing: About 63.94% done; ETC: 15:20 (0:02:33 remaining)
Discovered open port 49152/tcp on 10.10.10.40
SYN Stealth Scan Timing: About 70.82% done; ETC: 15:20 (0:02:04 remaining)
SYN Stealth Scan Timing: About 77.62% done; ETC: 15:20 (0:01:35 remaining)
SYN Stealth Scan Timing: About 84.25% done; ETC: 15:20 (0:01:07 remaining)
SYN Stealth Scan Timing: About 90.52% done; ETC: 15:20 (0:00:41 remaining)
Completed SYN Stealth Scan at 15:20, 433.31s elapsed (65535 total ports)
Initiating Service scan at 15:20
Scanning 9 services on 10.10.10.40
Service scan Timing: About 44.44% done; ETC: 15:22 (0:01:08 remaining)
Completed Service scan at 15:21, 59.58s elapsed (9 services on 1 host)
Initiating OS detection (try #1) against 10.10.10.40
Retrying OS detection (try #2) against 10.10.10.40
Initiating Traceroute at 15:21
Completed Traceroute at 15:21, 0.06s elapsed
Initiating Parallel DNS resolution of 2 hosts. at 15:21
Completed Parallel DNS resolution of 2 hosts. at 15:21, 0.08s elapsed
NSE: Script scanning 10.10.10.40.
Initiating NSE at 15:21
Completed NSE at 15:21, 9.86s elapsed
Initiating NSE at 15:21
Completed NSE at 15:21, 0.00s elapsed
Initiating NSE at 15:21
Completed NSE at 15:21, 0.00s elapsed
Nmap scan report for 10.10.10.40
Host is up (0.050s latency).
Not shown: 65526 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Aggressive OS guesses: Microsoft Windows 7 or Windows Server 2008 R2 (97%), Microsoft Windows Home Server 2011 (Windows Server 2008 R2) (96%), Microsoft Windows Server 2008 R2 SP1 (96%), Microsoft Windows Server 2008 SP1 (96%), Microsoft Windows 7 (96%), Microsoft Windows 7 SP0 - SP1 or Windows Server 2008 (96%), Microsoft Windows 7 SP0 - SP1, Windows Server 2008 SP1, Windows Server 2008 R2, Windows 8, or Windows 8.1 Update 1 (96%), Microsoft Windows 7 SP1 (96%), Microsoft Windows 7 Ultimate (96%), Microsoft Windows 8.1 (96%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 0.010 days (since Wed Mar 31 15:07:37 2021)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=259 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -15m26s, deviation: 34m37s, median: 4m32s
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-03-31T20:26:00+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-03-31T19:26:03
|_  start_date: 2021-03-31T19:12:25

TRACEROUTE (using port 5900/tcp)
HOP RTT      ADDRESS
1   48.18 ms 10.10.14.1
2   48.33 ms 10.10.10.40

NSE: Script Post-scanning.
Initiating NSE at 15:21
Completed NSE at 15:21, 0.00s elapsed
Initiating NSE at 15:21
Completed NSE at 15:21, 0.00s elapsed
Initiating NSE at 15:21
Completed NSE at 15:21, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 506.99 seconds
           Raw packets sent: 66637 (2.933MB) | Rcvd: 66292 (2.660MB)
```

- Nmap vuln scan 

```sh
nmap --script vuln 10.10.10.40 -p 135,139,445,49152-49157
```

```txt
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-31 15:40 EDT
Nmap scan report for 10.10.10.40
Host is up (0.051s latency).

PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 109.88 seconds
```

- Exploitation using Metasploit

![[bba1e3c2c14848ad0825630aab982863-1.png]]

- Finally, get the user.txt and root.txt from the respective user directories.

![[d6b7527719bf33dc3a01d12749daacb2-1.png]]

- Done!