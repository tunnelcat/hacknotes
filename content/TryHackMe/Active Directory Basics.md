# Forest Overview

![209fa4a3d244f33834f7261c2e2657cb.png](../../_resources/209fa4a3d244f33834f7261c2e2657cb-1.png)

Parts of a forest: 
- Trees - A hierarchy of domains in Active Directory Domain Services
- Domains - Used to group and manage objects 
- Organizational Units (OUs) - Containers for groups, computers, users, printers and other OUs
- Trusts - Allows users to access resources in other domains
- Objects - users, groups, printers, computers, shares
- Domain Services - DNS Server, LLMNR, IPv6
- Domain Schema - Rules for object creation

# Users + Groups

## Types of users

- Domain Admins - This is the big boss: they control the domains and are the only ones with access to the domain controller.
- Service Accounts (Can be Domain Admins) - These are for the most part never used except for service maintenance, they are required by Windows for services such as SQL to pair a service with a service account
- Local Administrators - These users can make changes to local machines as an administrator and may even be able to control other normal users, but they cannot access the domain controller
- Domain Users - These are your everyday users. They can log in on the machines they have the authorization to access and may have local administrator rights to machines depending on the organization.

## Types of groups

- Security Groups - These groups are used to specify permissions for a large number of users
- Distribution Groups - These groups are used to specify email distribution lists. As an attacker these groups are less beneficial to us but can still be beneficial in enumeration

## Default Security Groups (a lot...)

-   Domain Controllers - All domain controllers in the domain
-   Domain Guests - All domain guests
-   Domain Users - All domain users
-   Domain Computers - All workstations and servers joined to the domain
-   Domain Admins - Designated administrators of the domain
-   Enterprise Admins - Designated administrators of the enterprise
-   Schema Admins - Designated administrators of the schema
-   DNS Admins - DNS Administrators Group
-   DNS Update Proxy - DNS clients who are permitted to perform dynamic updates on behalf of some other clients (such as DHCP servers).
-   Allowed RODC Password Replication Group - Members in this group can have their passwords replicated to all read-only domain controllers in the domain
-   Group Policy Creator Owners - Members in this group can modify group policy for the domain
-   Denied RODC Password Replication Group - Members in this group cannot have their passwords replicated to any read-only domain controllers in the domain
-   Protected Users - Members of this group are afforded additional protections against authentication security threats. See http://go.microsoft.com/fwlink/?LinkId=298939 for more information.
-   Cert Publishers - Members of this group are permitted to publish certificates to the directory
-   Read-Only Domain Controllers - Members of this group are Read-Only Domain Controllers in the domain
-   Enterprise Read-Only Domain Controllers - Members of this group are Read-Only Domain Controllers in the enterprise
-   Key Admins - Members of this group can perform administrative actions on key objects within the domain.
-   Enterprise Key Admins - Members of this group can perform administrative actions on key objects within the forest.
-   Cloneable Domain Controllers - Members of this group that are domain controllers may be cloned.
-   RAS and IAS Servers - Servers in this group can access remote access properties of users

# Trusts + Policies
Trusts are a mechanism for users in the network to gain access to resources in the domain. 

![faa07b8fd7a50df7fb3e7f52155794fa.png](../../_resources/faa07b8fd7a50df7fb3e7f52155794fa-1.png)

## Types of trusts
-   Directional - The direction of the trust flows from a trusting domain to a trusted domain
-   Transitive - The trust relationship expands beyond just two domains to include other trusted domains

## Domain policies
There are many domain policies. Examples are: 
-   Disable Windows Defender - Disables windows defender across all machine on the domain
-   Digitally Sign Communication (Always) - Can disable or enable SMB signing on the domain controller

# AD DS + Authentication

## Domain services
There are many services that the domain controller can provide. Defaults are: 
-   LDAP - Lightweight Directory Access Protocol; provides communication between applications and directory services
-   Certificate Services - allows the domain controller to create, validate, and revoke public key certificates
-   DNS, LLMNR, NBT-NS - Domain Name Services for identifying IP hostnames

## Domain authentication
Two main types:
-   Kerberos - The default authentication service for Active Directory uses ticket-granting tickets and service tickets to authenticate users and give users access to other resources across the domain.
-   NTLM - default Windows authentication protocol uses an encrypted challenge/response protocol

# Azure AD
Cloud version of AD, acts as a middle man between physical Active Directory and user sign-on, making a lot of AD attacks ineffective.

![c7f8145d80a1c1206872469235b71c63.png](../../_resources/c7f8145d80a1c1206872469235b71c63-1.png)

| Windows Server AD      | Azure AD |
| ----------- | ----------- |
| LDAP      | Rest APIs       |
| NTLM   | OAuth/SAML        |
| Kerberos      | OpenID       |
| OU Tree   | Flat Structure        |
| Domains and Forests      | Tenants       |
| Trusts   | Guests        |

