# Quick Cheatsheet

| Option      | Description |
| ----------- | ----------- |
| `-sT`      | TCP Connect       |
| `-sS`      | SYN scan, "half-open"       |
| `-sU`   | UDP        |
| `-sN`   | TCP Null        |
| `-sF`   | TCP FIN        |
| `-sX`   | TCP Xmas        |
| `-O`      | Detect OS       |
| `-A`      | "All" scan, aggressive       |
| `-sV`   | Detect version        |
| `-vv`      | Double verbose       |
| `-oA`   | Save output in 3 formats        |
| `-oN`      | Save output in normal format       |
| `-oG`   | Save output in grepable format        |
| `-T5`   | Timing level 5, fastest, aggressive        |
| `-p-`      | Scan all ports       |
| `--script=vuln`   | Use Nmap vuln script category        |

# TCP Connect Scans
The `-sT` flag performs a three-way handshake to determine if ports are open/closed.

If a port is OPEN, the target will respond to the SYN with a SYN/ACK. Then Nmap completes the handshake by sending an ACK.

![a4c916a00574fef884a20c76da698b2d.png](../../_resources/a4c916a00574fef884a20c76da698b2d-1.png)

If a port is CLOSED, the target will respond to the SYN with an RST.

![971bd6bd35bae5b69557f9f662c363cf.png](../../_resources/971bd6bd35bae5b69557f9f662c363cf-1.png)

A port is considered to be FILTERED if there is no response, as packets are likely being dropped. 

# SYN Scans
Also known as a half-open or stealth scan. The `-sS` flag sends back a RST packet after receiving a SYN/ACK from the server (this prevents the server from repeatedly trying to make the request).

This is what an OPEN port looks like. 

![ed2ab48b1a4ae03e41aa8b5a9bafddd9.png](../../_resources/ed2ab48b1a4ae03e41aa8b5a9bafddd9-1.png)

CLOSED or FILTERED ports look similar to the TCP Connect scans.

Pros: 
- Stealthier
- Faster

Cons: 
- Needs sudo permissions (raw packet creation)
- Can bring down unstable services

# UDP Scans
UDP connections are stateless, so it is more difficult determining which ports are actually open. Lack of acknowledgement makes UDP much slower to scan.

When a packet is sent to an open UDP port, there should be NO response. Nmap refers to this state as `open|filtered`. If it gets a UDP response (unusual), then the port is `open`. If there is no response, a second request is made as a double-check. If there is still no response then the port is marked `open|filtered` and Nmap moves on.

When a packet is sent to a closed UDP port, the target response with an ICMP (ping) packet containing a message that the port is unreachable. This clearly identifies closed ports.

# NULL, FIN, and XMAS
These scans tend to be stealthier than even SYN "stealth" scans, mostly used for firewall evasion and such.

The `-sN` Null scan sends TCP requests with no flags at all. The target host should respond with a RST if the port is closed. 

![8aba32939a71b91a63255a6f95df3b33.png](../../_resources/8aba32939a71b91a63255a6f95df3b33-1.png)

The `-sF` FIN scan is almost identical, but sends a FIN flag instead of a completely empty packet (usually used to gracefully close active connections). Again, Nmap expects a RST if the port is closed. 

![778bda216916167278703fa8b16de326.png](../../_resources/778bda216916167278703fa8b16de326-1.png)

The `-sX` Xmas scan sends a malformed TCP packet and expects a RST response for closed ports. It's called Xmas because the flags that it sets (PSH, URG, and FIN) make it look like a blinking christmas tree in Wireshark. 

![cd5adadb2f5bb23b1e5c47fbb8fce12b.png](../../_resources/cd5adadb2f5bb23b1e5c47fbb8fce12b-1.png)

The expected response for open ports with these scans is also identical, and is very similar to that of a UDP scan. If the port is open then there is no response to the malformed packet. Therefore, ports are only identified as being `open|filtered`, `closed`, or `filtered`.

Note: while RFC 793 mandates that hosts respond to malformed packets with a RST TCP packet for closed ports, and don't respond at all for open ports; this is not always the case in practice. Windows and Cisco devices are known to respond with a RST to any malformed TCP packet. 

# ICMP Network Scanning
A ping sweep in nmap looks like: 

```
nmap -sn 192.168.0.0/24
nmap -sn 192.168.0.1-254
```

The `-sn` switch tells Nmap not to scan any ports, so it primarily uses ICMP echo packets (or ARP on local network). It also causes Nmap to send a TCP SYN packet to port 443 of the target, as well as a TCP ACK (or TCP SYN without root) packet to port 80.

Note that these ping sweeps are not 100% accurate but does provide a baseline.

# NSE Scripts
Nmap Scripting Engine (NSE) scripts extend functionality of Nmap. Scripts are written in Lua. 

Some useful categories: 
- `safe` - won't affect target
- `intrusive` - not safe, likely to affect target
- `vuln` - scan for vulns
- `exploit` - attempt to exploit
- `auth` - attempt to bypass authentication (e.g. anonymous FTP logon)
- `brute` - attempt to bruteforce creds
- `discovery` - attempt to query services for more info (e.g. query SNMP server)

Some scripts require arguments, and each are formatted like `<script-name>.<arg>` and are separated by commas: 

```bash
nmap -p 80 --script http-put --script-args http-put.url='/dav/shell.php',http-put.file='./shell.php'
```

Nmap scripts come with built-in help menus: 
```bash
nmap --script-help <script-name>
```

The help menus might not include all the arguments available. If you need more details, you can check the docs at https://nmap.org/nsedoc/

The default directory for Nmap scripts is `/usr/share/nmap/scripts`

Here are some ways to search for scripts (`script.db` is just a text file): 
```nmap
locate *.nse | grep ftp
grep "ftp" /usr/share/nmap/scripts/script.db
ls -l /usr/share/nmap/scripts/*ftp*
grep "safe" /usr/share/nmap/scripts/script.db
```

If you need to install the standard scripts, a `sudo apt install nmap` should be enough, but you can also load scripts manually (either your own or downloaded): 

```bash
sudo wget -O /usr/share/nmap/scripts/<script-name>.nse https://svn.nmap.org/nmap/scripts/<script-name>.nse
nmap --script-updatedb  # update the script.db file
```

# Firewall Evasion
Your typical Windows host firewall will block all ICMP packets. This means that Nmap will treat the host as dead and not scan it at all. You can override this with `-Pn`, treating it as alive no matter what. However, this comes at a cost of time because Nmap will try to scan and double-check potentially dead hosts.

Some switches for firewall evasion: 
| Option      | Description |
| ----------- | ----------- |
| `-f`      | Split packets into smaller fragments      |
| `--mtu <number>`      | Controls size of packets, must be multiple of 8       |
| `--scan-delay <time>ms`      | Add delay between packets       |
| `--badsum`      | Generates invalid checksum for packets (packet should be dropped normally, but firewalls may respond without checking, revealing themselves)       |

