# Takeaways
# Writeup
## Recon
- Used nmapAutomator to perform various recon scans

![[1b69ab4658f3509f3382739132031121-1.png]]

- Basic web enumeration

![[03c2e6c95196147dbf60d349accf134a-1.png]]

- phpbash is just a php webshell program (this would provide for initial access if found)

![[d3acfa6620c878b35c9ae92b49ba3684-1.png]]

- Nikto scan 

![[c4e775e70c107ad500f5b814a7dbf8c3-1.png]]

- Ffuf scan 

![[2d7d2ec5b9a360f2e11f56b4bea20ba1-1.png]]

## Initial Access

- The website suggests that phpbash might be located in the /dev/ folder, and it is!

![[e7878d0ec29debd923d600ae2b5cb3a1-1.png]]

- Clicking the phpbash.php link (the min version is just a small version of the same program) activates the shell, granting initial access as the web user

![[6f57b1b7a3e8ac87a97258eb4751ea85-1.png]]

## Privilege Escalation

- Basic enumeration of OS and sudo privileges

![[37d4d02de82a591363250889bbdebde7-1.png]]

![[c437c6b4bc68585f3b697f39cf4c60f1-1.png]]

- Basic enumeration of installed programs

![[98150f0c741719cc3ac8bcb9d565f8fa-1.png]]

- After a bit of experimentation with transferring files, it seems that the /var/www/html/uploads directory allowed arbitrary file upload with wget. This was exploited by uploading a php-reverse-shell (by pentestmonkey), catching the session with Metasploit (exploit/multi/handler), then upgrading to a Meterpreter session

- Modifying and serving the php reverse shell in preparation for file transfer

![[2da0b371dc817d53bb8e42803fd804e6-1.png]]

- Downloading the reverse shell on the target using wget

![[c3e62a663a7cc450263b50cc6354032a-1.png]]

- Using `use exploit/multi/handler` to catch the reverse connection (after navigating with the browser to the /uploads/reverse.php to activate the reverse shell)

![[377ff5c80962bf88b467510f35085372-1.png]]

- Backgrounded the basic shell with (CTRL+Z), then using post/multi/manage/shell_to_meterpreter module to upgrade the shell to Meterpreter

![[5aedad3003cc2459364ade08968a2ae4-1.png]]

- User flag

![[9827aed787c2db1ba740edb6cca7320e-1.png]]

- (One could make the case that this meterpreter maneuver was not necessary, but it was mostly for practice and the fact that a downgraded shell instance is better than the custom webshell provided anyways, since a python shell upgrade can be performed as follows) 

![[677f1977cb1d3565d4ca3c5a7e350175-1.png]]

- Note: the python shell now allows for `sudo` usage

- Browsing the root directory yields an interesting `/scripts/` directory with the following contents:

![[5d57eb4fc6c0443548227ecfe30b477e-1.png]]

- This indicates that the execute permissions on this directory for the current `www-data` user are insufficient (but read is permitted)

- However, since the www-data user can run sudo with scriptmanager's rights on any command, the same python shell as above can be used to elevate privileges to the scriptmanager user.

![[a24068cdc41a363aa956c9e2d451a7af-1.png]]

- The contents of /scripts/ is accessible to the scriptmanager user

![[7bfcdec51aa2df0790fdc7b07bd1a1a5-1.png]]

![[d9358cde678ce225eebecbfd38e6afa2-1.png]]

- What was interesting was that, although the test.py script was supposedly run with scriptmanager's permissions, it could not produce the test.txt file with root permissions. Considering also the date of the test.txt versus the test.py file, there seemed to be some scheduled tasks going on. That is, test.py seemed to run periodically with root permissions! This was only more suspicious when attempting to remove the test.txt file, as another one would take its place shortly!

- This could be exploited by modifying the test.py file to open a reverse shell connection back to the attacker machine, thus gaining a root shell.

- Using a custom python reverse shell to prepare for file upload

![[a5cc1636b5a77a240e3295b027ac093c-1.png]]
![[8e1941350124ef2d10598148b702b7e4-1.png]]


- Serving the file from Kali to download on target machine

![[a176fd58a724835d995c215f4ae9ab9e-1.png]]

- Downloading on the target machine

![[11e6a77923934199d6809359e4374c83-1.png]]

- Setting up a listener to catch the root shell

![[f4151b685eb04ff2e602e9208efd7e12-1.png]]

- Waiting...and root flag!

![[27d3222ea6d79eabd86fbcc1b113eafd-1.png]]
