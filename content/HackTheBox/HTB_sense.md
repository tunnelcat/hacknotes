# Writeup 
## Recon

### rustscan

```
sudo rustscan -a 10.10.10.60 | tee rustscan.txt
```

### ffuf scans

- I scanned for common files with common extensions with ffuf and found some interesting content, but I wanted to run a scan in the background too (since this one finished somewhat quickly)

```
ffuf -w /usr/share/wordlists/dirb/common.txt -u https://10.10.10.60/FUZZ -e .asp,.aspx,.php,.php3,.php4,.php5,.txt,.shtm,.shtml,.phtm,.phtml,.jhtml,.pl,.jsp,.cfm,.cfml,.py,.rb,.cfg,.zip,.pdf,.gz,.tar,.tar.gz,.tgz,.doc,.docx,.xls,.xlsx,.conf | tee ffuf-common.txt
```

![98d515f0869b08e5f6e129eb92348183.png](../../_resources/98d515f0869b08e5f6e129eb92348183-1.png)

- Tip: always have recon in the background. You never know when you might need it. In this case, I decided to run a ffuf on a medium-sized wordlist from SecLists along with some extensions I thought would be juicy for pfSense/Linux (notably, this scan eventually found `system-users.txt`, crucial for initial access)

```
ffuf -ic -t 100 -w /home/kali/HTB/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u https://10.10.10.60/FUZZ -e .php,.txt,.cfg,.conf | tee ffuf-directory-medium.txt
```

![edf3fec80f4eb431b123a402a1209ead.png](../../_resources/edf3fec80f4eb431b123a402a1209ead-1.png)

### Web enumeration

- From the login page, it is obvious that the webserver is running pfSense, an open-source firewall solution. Moreover, it looks dated (meaning vulnerabilities should be present...)

![7c99ae36453c16771e70ddf4b1cb00e1.png](../../_resources/7c99ae36453c16771e70ddf4b1cb00e1-1.png)

- It seems like this instance is running on top of DragonFlyBSD, a fork of FreeBSD. This is expected for pfSense.

![cecd1acac6379f52539a691f287b3728.png](../../_resources/cecd1acac6379f52539a691f287b3728-1.png)

- Now this is interesting. A changelog.txt doesn't seem standard (hinting at more hidden goodies with the `.txt` extension), and it reveals that ths system is not fully patched.

![e219d20c1a8c02cd5e535045c79d1d49.png](../../_resources/e219d20c1a8c02cd5e535045c79d1d49-1.png)

- With the larger ffuf scan running in the background, user credentials are found! But what's the password? 

![dc77507bcf51dcf9688fe62a370451b4.png](../../_resources/dc77507bcf51dcf9688fe62a370451b4-1.png)

### Logging in to pfsense with user account and default password

- As always, default credentials are easily searched online. I used the credentials `rohit:pfsense` (default pfSense password) to get access to the dashboard.

![09bf14b4a34bb5e9f3784f1398a41d16.png](../../_resources/09bf14b4a34bb5e9f3784f1398a41d16-1.png)

- Using searchsploit on the pfSense version leads to a suggestion that command injection may be possible

![b8def3e84a80196dc9f6319083295757.png](../../_resources/b8def3e84a80196dc9f6319083295757-1.png)

## Initial Access

### Using pfSense command injection vuln to gain shell

![693e775feb4b7f92044f2f22e2f527b2.png](../../_resources/693e775feb4b7f92044f2f22e2f527b2-1.png)

- Started a netcat listener on Kali to catch the shell

```
rlwrap nc -nvlp 1337
```

![9a269fae5767caa3634b7f8fcd31d62e.png](../../_resources/9a269fae5767caa3634b7f8fcd31d62e-1.png)

![e1349126853ce75e90f614aa2cf46e51.png](../../_resources/e1349126853ce75e90f614aa2cf46e51-1.png)

## Privesc

- Since the shell gives root, no privesc is necessary. User and root flag done! (the last # is part of the bugged shell prompt, not part of the user flag)

![764c77bb35afe29079e618092ea6cb1f.png](../../_resources/764c77bb35afe29079e618092ea6cb1f-1.png)