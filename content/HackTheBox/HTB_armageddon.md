# Takeaways
- ALWAYS look at default configuration files, then see if someone has changed them or made new configuration files. 
- Start by looking where you spawned if it seems even the slightest bit interesting. Maybe there are plaintext credentials. Remember: hide and seek.


# Process
- First, as always, a lazy nmap scan.

![[5e8b4931518dc6ed609d904ae06f2092-1.png]]

- Note: run nikto again. had to stop short.
- robots.txt is interesting, a lot of directories to unpack here.
- Could this be another mod_cgi exploit? 

![[467252e0b354f2455abe0f8833fb04c0-1.png]]

- Upon noticing the Drupal 7 install, I decided to search if there were any applicable exploits on exploit-db and found (keep in mind the name of this machine):

![[7e7e70e2e87f6423e7f68d0b4f880f98-1.png]]

![[976d0e49eb0636c87ab676539ec10303-1.png]]

- It turns out that Drupal is a mess...

![[e7fed326a9bf0becd6e82d411cfc4daf-1.png]]

- Using `msfconsole` (Metasploit) to exploit the Drupal vulnerability

![[68c5b7d3ae5405416cc69affe47a5ab5-1.png]]

![[3c582051d39adf60951e163562fe9265-1.png]]

![[bfd1b828f1159cdea23a873857d3bd70-1.png]]

- I need access to the home folder now. Privilege escalation should be the next step.

- Local Enumeration

![[9b7840c607aa3dbb4d0fb3bd43232c5e-1.png]]

- What is the next step here? I think enumeration definitely, but also what does the website have to do with the internals? I haven't gotten credentials yet, so it's kinda tough to say. 

- /etc/passwd file

![[a22e9191f9d914d5cc098cb4f9b1f8d1-1.png]]

- Network services

![[42e6278faa06fce70d5b3978928d2c82-1.png]]

- Finding "bruce"

![[f6331a3ffac6c489911906cf435a59d8-1.png]]

- After looking around for the LONGEST time (and a few hints), I was able to find that `/var/www/html/sites/default/settings.php` had credentials in plaintext for MySQL...

![[7ffecb46f50c4e93ec82e4244659ca84-1.png]]

- Logging in to MySQL and doing a bit of enumeration shows the `users` table in the `drupal` database.

![[053863657651d329827f4a893677da73-1.png]]

![[872d04c797c51ad2e232d7b920e37fbf-1.png]]

- Finally, after listing the entries in the table, we get the admin's password in a hash. 

![[83caaaf43bef5a17cc60d3fc84f865c6-1.png]]

- Googling the hash gives a probable hash type (hash identification)

![[c58b4cc42a7919c68eef102b4d02e216-1.png]]

- Time to crack the hash with hashcat.

![[34ab8672656c40934b95769e3415e777-1.png]]

- Done!

![[bb65a8f274e61da3bc56a5c18e9bb532-1.png]]

- Logging in with SSH and getting the user flag is now trivial using the cracked credentials.

![[e49196fbef3d9dd6bf439dff974337c2-1.png]]

- Now, onto root...
- Doing some basic enumeration with sudo reveals that snap can be run with root privileges as bruce

![[9153264327d1ea6f684b5699091fb399-1.png]]

- Trying to craft a malicious snap package and installing it seems like a good choice for privesc. 

- Using the information and Python script on https://github.com/initstring/dirty_sock/blob/master/dirty_sockv2.py, I made a custom Python script to decode the base64 and write a malicious `.snap` file called `exploit.snap`

![[b5c62593a407260a4cf5065d005c9928-1.png]]

- I uploaded the file to the target machine using curl and a Python HTTP server on Kali, and proceeded to install it with:

```sh
sudo snap install --devmode exploit.snap
```

![[31fb7bccd66b5c2763a107fa994ffb81-1.png]]

- Lo and behold, a new user was added! The credentials (known from exploit) are dirty_sock:dirty_sock 

- Now, all we need to do is to do a couple of su commands to get to root!

![[801dbc5bc64aaffd15a59fa035d79ead-1.png]]

- Done!

## Versions
Linux Kernel: 3.10.0-1160.6.1.el7.x86_64
GCC: 4.8.5 20150623 
Red Hat: 4.8.5-44
Apache: 2.4.6
PHP: 5.4.16
OpenSSH: 7.4
Sudo: 1.8.23
Drupal: 7.56(?)
Python2: 2.7.5
Python3: 3.6.8
MySQL: Ver 15.1 Distrib 5.5.68-MariaDB, for Linux (x86_64) using readline 5.1