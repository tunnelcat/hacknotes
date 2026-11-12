# Takeaways
- If it's supposed to be easy, just slow down, calm down, and think about simple ways to access things "hidden" in plain sight.
- Privilege escalation doesn't have to be complex (if misconfigured)

# Process
- First, a lazy nmap scan shows that HTTP, SSH, and FTP are open.

![[51514be519c5c054a53e480fdd2d1793-1.png]]

- The webapp looks like this: 

![[e0d307631321d302ca83fdebf5d19697-1.png]]

- The vsftpd version is vulnerable to a DOS attack, but this is not really useful in getting the user and root accounts.

- Instead, while playing around with the various links on the webapp, I found the security snapshot to be very interesting. In particular, it hosts data in the `/data/<number>` URL. 

![[f4513a34e644531c6c58313db47676e8-1.png]]

- Maybe "Nathan" has used this before? In that case, his data file would be closer to the beginning of the number sequence, and after navigating to `/data/0` and downloading the .pcap file and opening it in Wireshark...

![[81e9d02bf9e521a256840cf4e8152f59-1.png]]

- Plaintext credentials (nathan:Buck3tH4TF0RM3!) to the FTP server! In fact, since we're lucky (and nathan is lazy), these same credentials can be used to get into the server via SSH!

![[c157ddec4571452059ffc8336fe873d9-1.png]]

- After a bit of basic enumeration, I found that the `/var/www/html` directory was pretty interesting, particularly `app.py`

![[8dec805c8cc91c455b81c7e6e3d24b16-1.png]]

- The following code fragment means that we can run basically anything with uid 0, or root, privileges.

```py
command = f"""python3 -c 'import os; os.setuid(0); os.system("timeout 5 tcpdump -w {path} -i any host {ip}")'"""
os.system(command)
```

- So, fire up Python with: 

```sh
python3 
```

- Then, enter the commands to get the root flag: 

![[6a4eaad52c4e7ac2ca04a2ad6b2552a7-1.png]]

- Alternatively, just run `/bin/bash` as root:

![[38b7948d47c1750a6f11419cc8c9cf34-1.png]]

- Done!
- But why does this work? It turns out that this is due to Linux binary capabilities. The `setuid` is usually not allowed to run freely as root, but you can check binary capabilities (from the root directory) with: 

```
getcap -r / 2>/dev/null
```

![[29617b643ba0e5b68ac2bb5e45009e5e-1.png]]

- And because `setuid` is a capability, privilege escalation is possible!

![[e4f60b63aabd3c6c96b5c3b02b25e7c6-1.png]]