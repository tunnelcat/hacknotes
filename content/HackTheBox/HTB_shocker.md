# Takeaways
- cgi scripts are scary when Shellshock is possible
- Always look for processes that are running on root privilege. Maybe you can privesc

# Process
- First, as usual, the lazy nmap scan.

![[94cc5b85f2375c4be7d5ffecf2b792d1-1.png]]

- Then, an equally lazy nmap vuln scan...

![[661cf78b49ac095b33107a7549907bae-1.png]]

- Dirb scan on the webserver

![[70f0c1b66024341abd1d86f4a1f39df1-1.png]]

- Hmm, looks like there is an Apache instance. Maybe it is vulnerable to Shellshock? (Google the Apache version and look for exploit-db results...)

![[809e2ac271ed9c6e9a22c9ae012f043a-1.png]]

![[b0d1f50f5d426bc52918112839be9ab1-1.png]]

- Hmm, what to do now? Try harder...

![[726a600acf468c6f59c58ff62d239664-1.png]]

- Let's try to bruteforce cgi scripts in /cgi-bin/

![[e883c90adda272006f87ffc7852072ea-1.png]]

- Success! Now, use Metasploit to exploit the TARGETURI cgi script.

![[9d3402c83f455569b3081c0de4a75450-1.png]]

![[4e0616520c7de588ab84ac640c2a5af2-1.png]]

![[be34bd1a05dc4a257fb12e040913f66e-1.png]]

- User is now pwned! Now to get root...
- Interestingly, using `sudo -l` reveals that the shelly user has root privileges running perl. So the idea is to try to run bash from perl. 

![[f93f57f23eee37031eb4cb6ceb56da21-1.png]]

![[6de3c3c4aa16424b74b17b1f9365541f-1.png]]

- And with that, both root and user are pwned!