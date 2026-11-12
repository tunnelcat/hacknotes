# Takeaways
- Learn Powershell...
- Learn about tools for Windows privesc...

# Writeup
## Recon
- Using nmapAutomator.sh to do initial recon (automates nmap, nikto, ffuf, etc.)

![[5dcb5e6ae3232f3fb4850550ccad8666-1.png]]

- Nikto scan

![[da587b0201e536bccef00234659fb233-1.png]]

- Ffuf

![[54b1218afb9d76b68f930fb9527566ec-1.png]]

- Basic web enumeration

![[b65027abe3e904e38400602b24257dc5-1.png]]

- Looking for exploits related to HFS 2.3 yields promising results (RCE)

![[0b0af16e29c1e886098222f2bc8ea4ff-1.png]]

## Initial Access

- Modifying/using the remote command execution exploit to gain access

![[4f4cb6c4f3145b1a9ada8b5754f1e049-1.png]]

![[d9f2c7f28170a624e9058a5c7832285d-1.png]]

![[df8f701b2a46d2368eb747bcb9b25eff-1.png]]

- User flag found!

![[00bc2325c874b49e4ed6448100fbad90-1.png]]

## Privilege Escalation

- Some basic system enumeration with `systeminfo`

![[0367947f4e43c897a8656f056d5b7bd6-1.png]]

![[b7814a5b349e6aad07f312fceb4326dc-1.png]]

- Something interesting I found with winPEAS.exe (not really used)

![[6fe09b3f399a0064d876c78bfaa6861a-1.png]]

- At this point, I decided to use Metasploit to facilitate the privesc process. After some Googling (also could've loaded Sherlock.ps1), it seemed that MS16-032 might be worth a try since it was specifically for Windows Server 2012 R2 and because the number of cores on the machine was 2 (sufficient and necessary for the exploit to work)

![[e4739f10696d244f8bec15b8f4467e3c-1.png]]

- Redoing initial access with Metasploit

![[98ad4cf0facc869529d0a128f469eec8-1.png]]

- Initial user shell

![[5992f35e74df9063704e21c238e3a34f-1.png]]

![[2d671534ed969c831cce0f5f3605e426-1.png]]

- Backgrounding (ctrl+z) meterpreter session to use for ms16-032 post exploit

![[d01279f838dbf4c089dd571e2f52e413-1.png]]

- Setting SESSION of meterpreter shell and running ms16-032 privesc to get a SYSTEM shell

![[cbe55a0a4adb6c7752e464069c0e88d7-1.png]]

![[958d3774915c6a5e4d8feddc689f2f76-1.png]]

- Root flag found!

![[293a95bf5a72acafd61c9cf831065414-1.png]]
