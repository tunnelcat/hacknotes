# Writeup 
## Recon

### rustscan

- First, use rustscan to enumerate ports quickly.

```
sudo rustscan -a 10.10.10.84 | tee rustscan.txt
```

![[eac19f701e3dabf45f340bce55a81408-1.png]]

### nmapAutomator

- Let nmapAutomator run in the background for constant recon.

```
sudo nmapAutomator.sh -H 10.10.10.84 -t Recon 
```

- nmap script output

![[28c48dd4680a43e4c7b5ad1a29232b56-1.png]]

- nikto output

![[cda7c3bfffd8914b5cf35c9691250fba-1.png]]

- ffuf output

![[c0f91f8a9f4ea31c7438b1409d702bc5-1.png]]

### Initial web enumeration

- Visiting the root website leads to an interesting page with possible filenames and a form that fetches the specified file. 

![[6c477127eb2d92ec27281231aab41258-1.png]]

- Navigating to the various files leads to some good info about the machine.

![[25c098c414fb3577fc1c5e0f73d4d9e9-1.png]]

![[acad7d14a0808772198796aea38bd1d8-1.png]]

![[415ed439153bab79bc1e277e644d6b07-1.png]]

- Notably, in `listfiles.php`, there seems to be a `pwdbackup.txt` that could have a password in it. 

![[598e54d660b81a49889649fe8c454dc9-1.png]]


## Initial Access

### Testing for LFI

- First, since the script seems to fetch the files by path (maybe), I decided to test for some basic LFI (local file inclusion) vulnerabilities by fetching the `/etc/passwd` file.

![[8489ccd10cd25afee905b1327990917d-1.png]]

- It worked! Now we know that `charix` is the user to target for initial access.

![[9d45ceda3ccb23745cffc3d3ac792bd0-1.png]]

### Decoding base64 encoded password multiple times and getting SSH shell

- Now onto looking at `pwdbackup.txt`...

![[41062bfc721d3591bce554d6fb7cd26f-1.png]]

- The password is encoded in what looks like base64 encoding (notice the = padding at the end).

![[97ace8763c4c288ac71eee923597fd1a-1.png]]

- Decoding the string (copy-paste) multiple times on `base64decode.org` manually (could use other automated methods) eventually leads to the password:

```
Charix!2#4%6&8(0
```

- Connecting with SSH gives a user shell. User flag done!

![[69bed6b5409b85c8e35062529d862a2b-1.png]]

- But what is this `secret.zip`?

## Privesc

### Exfiltrating and examining secret.zip

- First, use `scp` to exfiltrate the zip file.

![[cbc6b8926b29a4b427308f7f90f6cc40-1.png]]

- Using the user's password (password reuse) allows extraction of the secret file. 

![[95b40ca9a3173b42b6e454930c5913fc-1.png]]

- Unfortunately, the file doesn't seem like much, so maybe something else is worth looking at. Perhaps this is a password file ("key") for something...

![[e69bf4167f52e2a8b3ce846d88a9b634-1.png]]

### Using LinEnum for recon and finding VNC

- Use `scp` to upload the LinEnum script. (could also use a number of other scripts)

![[f1f5cfa2d9183bab6b7f30cce743e4c9-1.png]]

- Run LinEnum and save the results to an output file. 

```
sh LinEnum.sh | tee out.txt
```

- Notice that an instance of VNC is running with root privileges...

![[743f6f5ef0192ec2cb63377453ac404f-1.png]]

### Port forwarding VNC and getting root using "secret" password file

- Since the system is running FreeBSD, `sockstat` allows enumeration of ports. It seems like VNC is using ports 5901 and 5801. 

![[34a249a4ab7b9468326ef331e8d50e7b-1.png]]

- Using SSH local port forwarding allows access to the local port in the target directly on a local port in Kali. In this case, I decided to try forwarding 5901 first to see if I could connect to it. 

```
ssh charix@10.10.10.84 -L 127.0.0.1:5901:127.0.0.1:5901
```

- Without a password, a connection with `vncviewer` is rejected, but using the `secret` file found earlier as the password file allows a graphical VNC connection!

![[33ef52b3ea3df68f0d6347d8bba334b0-1.png]]

- Root flag done!

![[99f6e1567764e1852d830670d3685f92-1.png]]