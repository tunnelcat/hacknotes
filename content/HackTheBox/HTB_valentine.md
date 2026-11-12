# Writeup 
## Recon

### rustscan

```
sudo rustscan -a 10.10.10.79 | tee rustscan.txt
```

![[10b8a526ec498b5a4dddbd4a66954eae-1.png]]

### nmapAutomator

```
sudo nmapAutomator.sh -H 10.10.10.79 -t Recon 
```

- sslscan output

![[103f47c0288950fec460532daa933c81-1.png]]

- nikto output

![[0d5cde9d01364f8e30d5c51517568f50-1.png]]

### Manual nmap scans

- Most notably, the nmap vulnerability scan shows that the server is vulnerable to Heartbleed, which should lead to information disclosure via a memory leak. 

```
sudo nmap -Pn --script vuln 10.10.10.79 -p80,443,22 | tee nmap-vuln.txt
```

![[92164739040f3d551a49627af772c062-1.png]]

### Initial web enumeration

- Looking at certificate for `https://10.10.10.79` reveals a potential hostname of `valentine.htb` (also found earlier in recon)

![[92d4a3b328575a8d88d0c5fbda484c9a-1.png]]

- Looking at the various web pages suggests something related to the Heartbleed vulnerability is key for initial access since the logo is displayed prominently.

![[ec3344f0b58e802b06f8892befe0e9e3-1.png]]

![[dd51402d6b5cd69e59298747b8142350-1.png]]

### Editing /etc/hosts to add valentine.htb

- Sometimes, visiting the website with a different hostname leads to a different site altogether due to virtual routing, so this step is just in case. 

![[065ecbe416eeb1ddea35cf511fdbfb99-1.png]]

### ffuf

- As always, I run ffuf to bruteforce interesting files and directories in the background. Always have recon in the background.

```
ffuf -ic -w /home/kali/HTB/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u https://valentine.htb/FUZZ -e .php,.txt,.shtm,.shtml,.cfg,.conf,.bak,.cgi | tee ffuf-directory-medium.txt
```

![[752832c54d7160aa671650fd21fde488-1.png]]

### Further web enumeration

- Since ffuf found `encode` and `decode` as potential files (and `/dev/` as a potential directory), might as well take a look.

![[a9e86ccd3aba6d0818ab4f846d19380e-1.png]]

![[01a2982ca4a821565664c3769d47a0d9-1.png]]

- Testing the encoder and decoder reveals that it just encodes and decodes in a base64 format...

![[0b4959ac2581d6a358bd2f8735b8a624-1.png]]

![[139dd04d239a1464eac22149261f5340-1.png]]

![[be22bb98a11371e9fae764b1a962b96e-1.png]]

![[5d421fd4c0ea218e8ed7b20565c836f8-1.png]]

![[1cd47023f15a8bc7ea7e4c239a63254b-1.png]]

- In the `/dev/` directory, there were some VERY interesting files. 

![[9557667acbbfc9e19dfbbf6a06e23e3f-1.png]]

- This file suggests that the encoding and decoding are done server-side, meaning that Heartbleed will expose anything entered into the encoder.

![[78fab43ed1f3601d13c236b7082cb83d-1.png]]

- This file, when decoded from hex (grouped in bytes), returns an encrypted RSA private key. When decrypted, this could be used to SSH into the server for initial access.

![[1396d726be9508d83eff48cb978fdc05-1.png]]

![[f24854d893dddb5166644a65c58cbb8c-1.png]]

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
+wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

## Initial Access

### Investigating Heartbleed

- Using searchsploit to search for Heartbleed exploits leads to two candidate Python files. (I don't want to deal with C if I can help it)

![[933851aa543803605483ee32050506b1-1.png]]

![[086ffc7d4aad11be2100f8431249df82-1.png]]

- Using `32764.py` multiple times (`32745.py` should do basically the same thing) eventually reveals some base64 encoded text that, when decoded, could be the password for the RSA private key found earlier!

```
python 32764.py valentine.htb | head -n 40   
```

![[87f3ba7df8eeb71ef285ad0203b9e18b-1.png]]

![[d978ebe9439c211ae7a134eec05eea94-1.png]]

- Fun fact: since the encoding directly happens on the server, the Heartbleed leak becomes obvious when you spam long strings of characters using the `encode.php` script. 

![[5b6f6e2c952d90a5b38b8afe74d0f7ff-1.png]]

### Decrypting the hype_key

- Save the hype_key somewhere (or create the file) and decrypt the key with the passphrase `heartbleedbelievethehype`

![[5c7c6ce64141fb700987ed45c0d32905-1.png]]

- The permissions for an SSH private key must be 600 (or 400).

![[f65a5762a147de197e2fe5c28fec0dad-1.png]]

### SSH login with hype_key

- While I didn't know the username to use for the SSH connection, one could reasonably guess that it COULD be `hype`, since it is referred to multiple times in the key, in the passphrase, etc. It turns out to be correct. User flag done!

![[4f23b5f531ce9a30942c97236e9388bf-1.png]]

## Privesc

### Basic enumeration

- As always, I check the `/etc/passwd` file for interesting info. Not much is here other than a possible password of "Hemorrhage" on the current user and the fact that many users have `/bin/sh` as a shell (which is weird maybe?)

![[9500a56df316746e43ba4dbb8e7d16a6-1.png]]

- I checked `.bash_history` in the home directory to see if there might be interesting commands. Indeed there is. Tmux is a terminal multiplexer, and this "dev session" may enable elevated privileges.

![[9f5cd9fd3949e767ee8eca3f2f19b3e5-1.png]]

- Looking at the `dev_sess` file, the owner is `root`, but users in the group `hype` can also use it, meaning our user can use the socket (denoted by the first "s") to login as root! If the owner:group was root:root, permission would be denied. 

![[72da72d7a1354ec2f06cc01206c94400-1.png]]

### Privilege escalation via tmux session

- Opening the tmux session with the `/.devs/dev_sess` socket elevates to root! Root flag done!

![[4aafd8e3ac1f3a222a70f88b6285f2f2-1.png]]

![[dff3c947c50a91fde8b218511a4bfa28-1.png]]


