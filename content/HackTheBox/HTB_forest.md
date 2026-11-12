# Writeup
## Recon

### nmapAutomator

![[91622777d6640b737cd31a6f8233dc96-1.png]]

![[ef58c386fe77869006517a7b454a20b2-1.png]]

![[f236eb0e395c3bf5e4ef23cae906503e-1.png]]

- The services highly suggest that the host is a Windows Active Directory Domain Controller of some sort (for one thing, multiple LDAP ports are listening).

### RustScan

![[5a846e4c472a34a2af1d299a821b93a6-1.png]]

### Using ldapsearch to enumerate users

- Before we can use ldapsearch to perform useful queries, the basedn (Base distinguished name)  must be enumerated

![[d1ad18c887609986090dbe2f482f4805-1.png]]

- Once the basedn is available, we can use it to search for usernames with the query `objectclass=Person sAMAccountName` and use `grep` to filter out just the usernames

![[334651b0ec8259b61c972df88a870eda-1.png]]

- The Impacket GetNPUsers.py module was able to find a user and password hash!! You can either use this module by installing from the git repository (Google it), or just use the `impacket-GetNPUsers` command from the `impacket-scripts` Kali Linux package

![[9a3db9eb217e6f2fdbf2892424879484-1.png]]


## Initial Access

### Hashcat cracking (Kerberos 5, etype 23, AS-REP)
- With the user `svc-alfresco` and password hash enumerated, we start hashcat to get credentials

```
Hash: $krb5asrep$23$svc-alfresco@HTB.LOCAL:0b586871b0b77326d23f783a69a2158d$e3c4b39251923604043e4c51be99f487c4b5b2039e8fb77353a038a9dfe69ed3037dd9e5f70770d78d757893b36a3ad374f5dd62f6d744300cff13160c28def850995448c257a1dadba39223058999b5d9e3b8cc42c9227a0570dae417169b6fd4142067ac81ffd0226c060bd56bac9a42ddc20ae9384d7895c4f5a4f4a8d3ce17ee9af0f0fcd35b0eb40940a4eb9603090bfc886ee0f38d7e40075e31ced1bb68169fde08f8bb88472dcd8fff260b8acd1848f1e489f32314560bdfcab9b6b306cd362249d971a7d6ef7665459aafe7e027c56d938ff51dae447d973304524ddf191b33d805
```

- Searching hashcat for the hashtype with vim search for `$krb5asrep$23`

```
hashcat --example-hashes | less
```

![[47843f841d88f802c5eeb7f40cab8c7e-1.png]]

- So the hashtype is 18200 and we can begin cracking

![[e36826273ad8d50e47a63786b1697f9f-1.png]]

- The cracked credentials:  `svc-alfresco:s3rvice`

![[778c3fa3660fc1c8c223e1a83964c77f-1.png]]

### Evil-WinRM to get a user shell
- Since port 5985/tcp (winrm) is open, we can use Evil-WinRM to get a basic user shell with the credentials

![[b8e004f2023cd7cf26facb41e8d36fc6-1.png]]

- User flag done!

![[a8804a2cbf3af777e9352edee7b9a59a-1.png]]


## Privesc

### Running WinPEAS remotely using impacket-smbserver
- While you could upload winpeas and execute it locally on the machine, using impacket-smbserver is a little stealthier
- The first step is to start impacket-smbserver in the directory of the winpeas x64 binary (exe) with dummy credentials

![[b4aa6c517cf13321cc8a2a7d048d17cf-1.png]]

- Using the WinRM shell from earlier, we connect to the smb share by performing various steps for authentication with the dummy credentials we created
- Reminder: the IP address should match the one for the HackTheBox VPN interface (tun0) 

![[3b27de18107d049e20de45b97d374015-1.png]]

![[33e023e1fa57429307deae99ccc113fd-1.png]]

- Accessing mydrive and running winpeas with the log option

![[10d0d1f65e85db0b0c4a4e5d6dadd63e-1.png]]

### Running BloodHound on the Domain Controller
- Since we are using Kali, we need the linux-x64 Release of BloodHound (download from Github) and an installation of neo4j (in the Kali repository)
- Once everything is downloaded and ready, upload SharpHound.exe to the smb share folder used earlier and run it on the target machine

![[7d0364de21f276bce2324617dc4d7a83-1.png]]

- Back on Kali, start up neo4j and navigate to the web interface with a web browser to login and change the password if needed

![[83d47856dd041ea99b1e8036e7b67303-1.png]]

- On Kali, run BloodHound, and once the SharpHound collection finishes, drag and drop the zip file to BloodHound for analysis

![[fbea08db07ab2cea6f4a10ca6d73db2d-1.png]]

![[9c664ce3f2de2f06ece769d798290546-1.png]]

![[6f4131b00c9696b3dd238d37a05331fd-1.png]]

- In the search bar, search for `svc-alfresco`, right click, and mark the user as owned. Then use a pre-built analytics query to find shortest paths from the owned user to admin

![[b5fe0990c4a1592b0dcee438b3c9d388-1.png]]

![[f3f2b67e548ee0ebe18b156ec3f9eb37-1.png]]

![[cf9522d761da1bc0909a71b908adaacd-1.png]]

### Adding a user for privesc
- We can see that svc-alfresco is a member of Account Operators, so first we need to create an account, add it to the Exchange Windows Permissions group, then escalate the user with privexchange

![[8c29c86f34617db174ec1e87a64150c5-1.png]]

![[0f498bcb275c3814b3b7cbc20e2a94df-1.png]]


### Escalating a user through PrivExchange using ntlmrelayx
- Specify the LDAP server and user to escalate

![[746ce4ad5b8780ebc4da109bcf944d70-1.png]]

- Navigate to the web interface and login as the user 

![[82411cfc55c092fffc62f8636629da19-1.png]]

![[2df62deff80c5209b7cfbb752c052570-1.png]]

![[93802f08b77d0725f9bdbaca970daec7-1.png]]

### Using secretsdump to get the admin hash and getting admin with psexec
- Finally, with the escalated user creds, use secretsdump to dump the NTDS.DIT secrets for hashes

![[3571c7dd1d464782bf939f043e5e819a-1.png]]

- And use psexec to login as administrator using the hash

![[5e8ecf04b2b745906b10e3ed5f0118f0-1.png]]

- Root flag done!

![[92afc9f8b6b867509b99189c85099d37-1.png]]

## Privesc (failed attempt)

### Abusing WriteDacl with PowerView (Add-DomainObjectAcl)
- We can see that svc-alfresco is a member of Account Operators, so first we need to create an account, add it to the Exchange Windows Permissions group, then use the abuse info (right click on a graph edge -> help -> abuse info) on the WriteDacl to gain access to the Administrator account credential hash

![[e41b2a5413ab2c1245278d4ae1cc217e-1.png]]

![[8c29c86f34617db174ec1e87a64150c5-1.png]]

![[0f498bcb275c3814b3b7cbc20e2a94df-1.png]]

- To abuse the WriteDacl, PowerView is required (included in PowerSploit repository). Run a python http server on port 80 and execute the script on the target machine using IEX

![[2678f63d68cd866bca4e55c9a03635a0-1.png]]

- Give the created user DCSync rights using Add-DomainObjectAcl (syntax in BloodHound doesn't work so Google an alternative way)

![[fa0abc7673a92104e74a205654e2caf7-1.png]]