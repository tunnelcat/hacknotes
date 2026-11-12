# Takeaways
- VPN connections means that you have to use the VPN interface IP if you want to set up a shell/reverse shell. Otherwise, the vulnerable machine will not be able to see the IP you use. 

# Process
- First step is just a basic enumeration nmap scan

![955617655ce1967db171b8e5d3c4e5fe.png](../../_resources/955617655ce1967db171b8e5d3c4e5fe-1.png)

- Next up is a basic vulnerability scan with nmap

![f7238f3f91f6c9fc9ed7028a6600e865.png](../../_resources/f7238f3f91f6c9fc9ed7028a6600e865-1.png)

- Looks like this system is some sort of Windows XP machine vulnerable to MS17-010. Using Metasploit, we can confirm again that it is indeed vulnerable.

![e9740ee96d7c2e6ab3e601dd15fff2d3.png](../../_resources/e9740ee96d7c2e6ab3e601dd15fff2d3-1.png)

![21df6bfbf7a35d3c61c3429a06e4c226.png](../../_resources/21df6bfbf7a35d3c61c3429a06e4c226-1.png)

- SMB enumeration...

https://0xdf.gitlab.io/2018/12/02/pwk-notes-smb-enumeration-checklist-update1.html

![b1921255031825e5a9428820074449e6.png](../../_resources/b1921255031825e5a9428820074449e6-1.png)

![cb3ce0077f5744bd8baceb6a890a235f.png](../../_resources/cb3ce0077f5744bd8baceb6a890a235f-1.png)

- OK, time to get serious and try the Metasploit exploits for MS17-010.

![e3f3d13d509ec92a8e7e07d1041d7665.png](../../_resources/e3f3d13d509ec92a8e7e07d1041d7665-1.png)

- Using the info command and some reading, it seems that psexec is the one to use for Windows XP since it supports x86 architecture. 
- VERY IMPORTANT: when setting up a reverse shell, we need to be mindful that we are using a VPN connection to access the HTB virtual machine. This means that we need to find our own IP on the VPN interface with ifconfig. 

![4bf7d7443beecf23392ee0e86e851c2b.png](../../_resources/4bf7d7443beecf23392ee0e86e851c2b-1.png)

- In this case, the tun0 interface is the VPN, and our IP address is 10.10.14.6
- Now continue with the exploit.

![7e19ba9868e4691cd117f2bcbab5599d.png](../../_resources/7e19ba9868e4691cd117f2bcbab5599d-1.png)

- Setting payload to regular reverse_tcp shell for compatibility, and set RHOSTS to target IP, LHOST to Kali machine VPN interface IP

![433c6ee0f13da555e514063b191d5612.png](../../_resources/433c6ee0f13da555e514063b191d5612-1.png)

![94f5d2d7a90b257899c242e3e3e6118c.png](../../_resources/94f5d2d7a90b257899c242e3e3e6118c-1.png)

![42402e07bb6c6c9b8835544477c7b06c.png](../../_resources/42402e07bb6c6c9b8835544477c7b06c-1.png)

- Run exploit

![cd6d84993288a15d715ea6ba2a6c19fb.png](../../_resources/cd6d84993288a15d715ea6ba2a6c19fb-1.png)

- Great! Now, let's find the user and admin Desktop folders to find the flags (This is usually where HTB boxes store the flags)

![ccebf03b998d05709ea635cd4f229373.png](../../_resources/ccebf03b998d05709ea635cd4f229373-1.png)

![adc8dd88a03655c7c7c35467f0cf19c9.png](../../_resources/adc8dd88a03655c7c7c35467f0cf19c9-1.png)

![c47b5dd619218d6b24bbb4d21bc3c62e.png](../../_resources/c47b5dd619218d6b24bbb4d21bc3c62e-1.png)

![b1a0a1357b5ad80f399d94b73acf818b.png](../../_resources/b1a0a1357b5ad80f399d94b73acf818b-1.png)

- Done!