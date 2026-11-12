# Takeaways
- USE GOOGLE! (And start simpler with versions and stuff before going into deep searches)

# Writeup
## Recon
- Recon scan with nmapAutomator (automates nmap, nikto, ffuf, and others)

![[0b0e89d61135b8bd16936e92250f8003-1.png]]

- Found ports 21, 80 open. Also anonymous FTP login with some interesting files (notably shell.aspx)

![[4d15f198d2b23fdd8c034a40e816441a-1.png]]

- Nikto scan

![[c3762ad030e4f9187b60f3ead2644092-1.png]]

- Ffuf scan 

![[cfa8290c5c88407409133337aeaaecf0-1.png]]

- Basic web navigation reveals a default-looking IIS 7 install

![[4381952146f8e1828245dd349d26767d-1.png]]

## Initial Access
- Used the anonymous ftp login (credentials anonymous:anonymous) and ran `get shell.aspx` to download the interesting file and examine it. It turned out to be a reverse shell, and it was easily modified and uploaded (`put shell.aspx`) using FTP to gain initial access to the target. 

- Getting the IP of the attacker machine for the reverse shell (10.10.14.5)

![[3c784a4a8516b7fb3fde8372b257dac7-1.png]]

- Modifying the shell.aspx for reverse shell preparation

![[eb0b89f4a794bd7f83c0f91f064807c7-1.png]]

- Netcat listener

![[f60abd48b6c331c24515042ce4ce01bd-1.png]]

- Uploading the modified shell.aspx with FTP 

![[5c02cc488a59026ebc17908e4f9d614d-1.png]]

- Activating the shell by navigating to the uploaded file with a web browser

![[5b9206a0c54295032ed80bdaf47f2c29-1.png]]

- Shell session opened

![[e8ddaf65035b4e0113e73c955f602dd2-1.png]]

## Privilege Escalation
- Running `whoami` shows that the shell opened with permissions of the web user

![[15d2873a764650ceb2b2b96e4ce47711-1.png]]

- Additional recon reveals the user folders for Administrator and babis, but the current shell has insufficient privileges

![[418a1544a887aefb1c43927f0110097b-1.png]]

- Basic Windows enumeration (also shows the babis user)

![[acd9a728e13ffcc5daed2cbf2a1256fd-1.png]]

- System architecture

![[977359c4d52a2b78b6c9bb855caad257-1.png]]

- Finding where the uploaded shell.aspx was stored

![[be51302a2ea201b093e3167a77c35f91-1.png]]

- Uploading winpeas.bat with the FTP login

![[0340a482c027e5c1d2bbca798dfe3c54-1.png]]

- Running winpeas.bat

![[fe8b9f0592ec12bbb080fe89ef283fcd-1.png]]

- Not many really intersting results, but a Google search of privesc exploits available for the current OS reveals that it may be vulnerable to MS11-046.

![[85e56c0e3994659a4d08d0d5444bfa4b-1.png]]

![[a5740635b7eb7e7509bc17722057a8f5-1.png]]

- Checking the security patches installed (it seems none were applied at all)

![[79dbb205d1c1330a82311d6a4da18d4f-1.png]]

- Getting the local privesc exploit and uploading it to the target

![[1f1b10631f1dc1fdb292038746903a3d-1.png]]

- On second thought, it may be possible (and faster, stealthier) to compile on Kali (using a cross-compiler like mingw-w64) and then deliver the executable 

![[cced4f0fb856fdac3f113df1ab19e72c-1.png]]

- As per the notes included in the exploit, compiling on Kali for 32-bit Windows

![[e5f120aa7eae1ddf120f05fa211cbf5c-1.png]]

- Uploading the exploit with FTP (note the required binary command/flag)

![[095b5d06d0068875ed0f4d854d74cafb-1.png]]

- Running the exploit on the target machine

![[b321a13fca2350ba7dad3b65fb59603a-1.png]]

- Root flag

![[36341071a71130e1088260a697188307-1.png]]

- User flag

![[4a71e516980a303553777c62af1ed4d9-1.png]]

- Cleanup (some files were from another user using the same box)

![[77767f5140777ad3558a20bba30c8fe5-1.png]]

- Note: before deleting the MS11-046.exe exploit, the SYSTEM shell must be closed

![[0c7e2f2f062b732b5f67f9c1e313d4e1-1.png]]