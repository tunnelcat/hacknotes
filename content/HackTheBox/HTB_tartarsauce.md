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

### ffuf

```
ffuf -ic -t 100 -w /home/kali/HTB/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.10.10.88/FUZZ -e .php,.txt,.cfg,.conf | tee ffuf-directory-medium.txt
```

### Web enumeration

![872b554531f07f341b544c798ba1219b.png](../../_resources/872b554531f07f341b544c798ba1219b-1.png)

![9cb7e1ead0754870ed24bd069ad541eb.png](../../_resources/9cb7e1ead0754870ed24bd069ad541eb-1.png)

- There is a link on the monstra page to an admin panel. 

![a58047feb330e31b8b4a533dc76fd6d1.png](../../_resources/a58047feb330e31b8b4a533dc76fd6d1-1.png)

- The other sites return a 404 Not Found error

### Vulnerability recon 

![36f5dda512fa8df8c9878009746b9120.png](../../_resources/36f5dda512fa8df8c9878009746b9120-1.png)

![e2e8aa2dcd52253e8d3c7c9279190ab3.png](../../_resources/e2e8aa2dcd52253e8d3c7c9279190ab3-1.png)

### Further web enumeration

![f3009259dfab35a65576a38f95d99cb7.png](../../_resources/f3009259dfab35a65576a38f95d99cb7-1.png)



## Initial Access


## Privesc