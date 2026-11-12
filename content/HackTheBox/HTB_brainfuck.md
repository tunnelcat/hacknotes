# Takeaways
- Check certificate information in browser if available.
- "Poisoning" your own /etc/hosts to resolve DNS for websites that are not really on the Internet, but (for the case of VMs, for example) "should" be. 
	- See: virtual routing
	- https://medium.com/@w3rallmachines/poisoning-your-hosts-file-a-guide-6e810b852538

# Fails
- Tried EXIM and Dovecot command injection in Metasploit (nope.) https://www.rapid7.com/db/modules/exploit/linux/smtp/exim4_dovecot_exec/
- Tried https://www.exploit-db.com/exploits/34896, Postfix Shellshock Remote Command Injection. Not working?

![47acc12efce26e636157a8692d1d3e29.png](../../_resources/47acc12efce26e636157a8692d1d3e29-1.png)


# Process
- First, nmap on 10.10.10.17 for basic enumeration

![36228dad0f8ffb31125191872c84e04b.png](../../_resources/36228dad0f8ffb31125191872c84e04b-1.png)

- seems to be a mail server, with a web server??

- Checking out the certificate thru browser... note the orestis@brainfuck.htb email

![fc7484e9492d9df6524bf387f3a24e66.png](../../_resources/fc7484e9492d9df6524bf387f3a24e66-1.png)

- Editing the /etc/hosts file to resolve the URLs with provided DNS Names (virtual routing)

![3789e7e6e061bf655c94282c94c2b5dd.png](../../_resources/3789e7e6e061bf655c94282c94c2b5dd-1.png)

- Visiting the website @ https://sup3rs3cr3t.brainfuck.htb/

![f4bed0abc686cd1b6a556e7b365aa3c8.png](../../_resources/f4bed0abc686cd1b6a556e7b365aa3c8-1.png)

- Visiting the website @ https://brainfuck.htb

![a79d6f877cfe27254953cbfc02531a56.png](../../_resources/a79d6f877cfe27254953cbfc02531a56-1.png)

- brainfuck.htb is WordPress. You know what that means! (wpscan time)

![db02491c516553174ccfc7473a92def2.png](../../_resources/db02491c516553174ccfc7473a92def2-1.png)

- Interesting: XML-RPC is enabled. There might be a way to use this to brute force the credentials w/ system.multicall()

![4859f982eb2fa0a4c45f64e1038453b3.png](../../_resources/4859f982eb2fa0a4c45f64e1038453b3-1.png)

- What are the valid accounts? Testing invalid users/passwords with brainfuck.htb/wp-login.php, also with wpscan enumerate option: 

![a86aa466defee956b5b0ba465785ce4e.png](../../_resources/a86aa466defee956b5b0ba465785ce4e-1.png)

![4b4cd05fe6cf928cfae19565698d08cb.png](../../_resources/4b4cd05fe6cf928cfae19565698d08cb-1.png)

![9b3637be414ab5b8664a3555de80d765.png](../../_resources/9b3637be414ab5b8664a3555de80d765-1.png)

![c3acb49be0a25b870eedfa00d7ba5c04.png](../../_resources/c3acb49be0a25b870eedfa00d7ba5c04-1.png)

![8a716677cd2884de950ec10bcd3fd020.png](../../_resources/8a716677cd2884de950ec10bcd3fd020-1.png)

- Bruteforcing proved to be mitigated by some request limits... Time to find another way in. 

- Remember that vulnerable plugin detected by wpscan? Might be interesting to check out publicly available exploits...

![8b74f57b6a23e4717cd6025f8a3096bb.png](../../_resources/8b74f57b6a23e4717cd6025f8a3096bb-1.png)

- Looks like there is an unauthenticated login exploit on exploit-db. 

![5739ad8d8d1722ae3b392b0ecf6d2c57.png](../../_resources/5739ad8d8d1722ae3b392b0ecf6d2c57-1.png)

- Unfortunately, the URL doesn't work on brainfuck.htb

![73e47fdb928ead5a7c011e5ac7a28df7.png](../../_resources/73e47fdb928ead5a7c011e5ac7a28df7-1.png)

- Or does it? Looking at the "Open Ticket" page, there seems to be some interesting code...

![05182d750633361cde78ae62577c1939.png](../../_resources/05182d750633361cde78ae62577c1939-1.png)

![fbe38d6677b25c8005fa1e22bdf2d2b1.png](../../_resources/fbe38d6677b25c8005fa1e22bdf2d2b1-1.png)

![6322102b4e9f0ed4e169e41a2f58da6f.png](../../_resources/6322102b4e9f0ed4e169e41a2f58da6f-1.png)

- Navigating to the admin-ajax.php URL by copy-pasting, it looks like the URL works with double slashes...

![31ac9807d73905fca25ec33dbd6465ce.png](../../_resources/31ac9807d73905fca25ec33dbd6465ce-1.png)

- Now, make a new .html file containing the exploit code and modify the URL

![57593037c89e7b412e02f99aeb9837f0.png](../../_resources/57593037c89e7b412e02f99aeb9837f0-1.png)

- Opening the .html exploit, we can login to any user. Pick admin. 

![310e59a6b80b7354ffcb98d907d496fc.png](../../_resources/310e59a6b80b7354ffcb98d907d496fc-1.png)

- Now that we have the session cookie, it's time to see what has changed at brainfuck.htb...

![4545b0158b369b74c2b550f34a3bdda5.png](../../_resources/4545b0158b369b74c2b550f34a3bdda5-1.png)

- Looks like we have admin on the WordPress site. Let's check out the plugins. 

![7a6281a64f5a6a8bee902bc5a2e80bb8.png](../../_resources/7a6281a64f5a6a8bee902bc5a2e80bb8-1.png)

- We know about the WP Support Plus since we used an exploit on it, but Easy WP SMTP is interesting since we know the machine also hosts some sort of mail server from initial enumeration. Let's take a look at the settings for that plugin. 

![e54c43f5961ed664108beecc525a1c1c.png](../../_resources/e54c43f5961ed664108beecc525a1c1c-1.png)

- Revealing the password for the SMTP authentication with HTML/Inspect Element...

![8482fc04e99642a5a9729284978e1640.png](../../_resources/8482fc04e99642a5a9729284978e1640-1.png)

- Let's put that account to good use! Remember that Dovecot instance in the beginning? We can use the power of POP to read emails via the "orestis" user.

![25077f785b687001ff088a58352f3ce6.png](../../_resources/25077f785b687001ff088a58352f3ce6-1.png)

- Reading email id 2 gives us credentials for the secret forum!

![a39bcca08e5b66f29685be356388fc6f.png](../../_resources/a39bcca08e5b66f29685be356388fc6f-1.png)

![9e8c8af7611f676873283f966390aadc.png](../../_resources/9e8c8af7611f676873283f966390aadc-1.png)

- As orestis, we now get access to interesting "secret" threads.

![5d0632af114a9bb03c9948a851ed1af3.png](../../_resources/5d0632af114a9bb03c9948a851ed1af3-1.png)

![c11928a450d1b2e4c56636152dc83ce9.png](../../_resources/c11928a450d1b2e4c56636152dc83ce9-1.png)

![57d2e8a957eb3e3e0cd379cb51386181.png](../../_resources/57d2e8a957eb3e3e0cd379cb51386181-1.png)

![1efbe0b18839f7db73da3b5931ddfc36.png](../../_resources/1efbe0b18839f7db73da3b5931ddfc36-1.png)

- Here is the full transcript: 

```txt
Mya qutf de buj otv rms dy srd vkdof :)

Pieagnm - Jkoijeg nbw zwx mle grwsnn
-----------------
Xua zxcbje iai c leer nzgpg ii uy...
-----------------
Ufgoqcbje....

Wejmvse - Fbtkqal zqb rso rnl cwihsf
-----------------
Ybgbq wpl gw lto udgnju fcpp, C jybc zfu zrryolqp zfuz xjs rkeqxfrl ojwceec J uovg :)

mnvze://10.10.10.17/8zb5ra10m915218697q1h658wfoq0zc8/frmfycu/sp_ptr
-----------------
Si rbazmvm, Q'yq vtefc gfrkr nn ;)

Qbqquzs - Pnhekxs dpi fca fhf zdmgzt
```

- Already I notice that, every time Orestis posts, he ends with "Orestis - Hacking for fun and profit". We can use this to crack the cipher. 

![a610fc592bf2927baee1f0cd331684fa.png](../../_resources/a610fc592bf2927baee1f0cd331684fa-1.png)

![42c4fda98312d8f977ef7bdc8aaee7f3.png](../../_resources/42c4fda98312d8f977ef7bdc8aaee7f3-1.png)

- So, the key is just a sequence of simple "Caesarian" alphabetic offsets. Once the whole key is used, it just repeats in a cycle. So, we just need to figure out where the key starts. It seems that the non-alphabetic characters are ignored and an offset is not applied at all to such characters. 

![90f3c526b9caae6aadb6dd656f6cc5ff.png](../../_resources/90f3c526b9caae6aadb6dd656f6cc5ff-1.png)

- Using this (the above) message to figure out where the key starts is trivial. The first character of the second line is encrypted with an offset of 8. Then we count the number of previous characters (not including any spaces or non-alphabetic characters) and figure out what should be the appropriate offset number for the first character in the first line using the known cyclic key. Going backwards, we can see that the first offset used is 5, so the key likely begins with 5 for every message. Of course, we could've brute forced it too, since the key is so short. 

- This code decodes all the messages in the thread using the known cyclic key. Again, the key just enables alphabetic offset encryption, which is trivial to decrypt (just do the opposite)

![7ef2b64e924463b65efe950a931ad5a3.png](../../_resources/7ef2b64e924463b65efe950a931ad5a3-1.png)

![db14c08fd7680f1b9b7fcc0d5e6790e4.png](../../_resources/db14c08fd7680f1b9b7fcc0d5e6790e4-1.png)

- Now we get the encrypted id_rsa SSH key. To bruteforce the password, first convert it to a hash that john (the ripper) can use, then feed that to john along with a wordlist (I'll be using rockyou.txt, a classic)

![36e9865a37c3ebbd08d2525e967ad8bb.png](../../_resources/36e9865a37c3ebbd08d2525e967ad8bb-1.png)

![b364919d36120bd3ca6cfdbedbc325a7.png](../../_resources/b364919d36120bd3ca6cfdbedbc325a7-1.png)

![847b0b49fad85b54c7de540b8e6e4870.png](../../_resources/847b0b49fad85b54c7de540b8e6e4870-1.png)

![63914e7f18d44cbfe87747f7460285e5.png](../../_resources/63914e7f18d44cbfe87747f7460285e5-1.png)

- Hooray, we have pwned user!

![d5ae5c15196b27cf404cdb299d03058f.png](../../_resources/d5ae5c15196b27cf404cdb299d03058f-1.png)

- Onto root... There are a couple of interesting files. Namely:
- debug.txt

![4815458942d2ba66a593e0d2ea824027.png](../../_resources/4815458942d2ba66a593e0d2ea824027-1.png)

- encrypt.sage
	
![03085ffdc3b601ff0593fec957a713b7.png](../../_resources/03085ffdc3b601ff0593fec957a713b7-1.png)
	
- output.txt

![6e65137934e3462f4a3d4952f659ef8d.png](../../_resources/6e65137934e3462f4a3d4952f659ef8d-1.png)

- It looks like root's encrypted password is in output.txt. But how do we decrypt it? 
- Looking at encrypt.sage (the algorithm used to encrypt the password), turns out this encryption algorithm is the RSA algorithm. Some number theory ahead. 
- In encrypt.sage we can see that root's password (with readable characters) is first converted to hex, then converted to a large integer. This is then used as the encrypted ciphertext for the RSA algorithm. Thus, we will need to reverse the process once we get the plaintext as a large integer. 
- Remember, we have p, q, and e in the debug.txt file. These will be useful in the decryption process. 
- Let m be the message to be encrypted (in this case, it's the password we want to get...)
- c = m^e mod n, where c is the encrypted ciphertext, and n = p * q.
- Then, m = c^d mod n.
- But, what is d? and how can we get it? d is basically the "modular inverse" of e. There exists an "Extended Euclidean Algorithm" that can calculate this given e and phi = (p-1) * (q-1). 
- Thus, pasting in the standard algorithm to get the modular inverse and the required values for p, q, e, and c (ciphertext), we can easily decrypt m, the encrypted password.

![b83f6b3309f4a5ebf51c36d2730b051f.png](../../_resources/b83f6b3309f4a5ebf51c36d2730b051f-1.png)

- At the end of everything, we get a large integer as plaintext. We can convert it to hex, then convert it to ASCII (readable characters) to get flag for root.

![44452b3175eff2a659e49c9765abfce4.png](../../_resources/44452b3175eff2a659e49c9765abfce4-1.png)

![848e7b2436942df5c2eac810a725b739.png](../../_resources/848e7b2436942df5c2eac810a725b739-1.png)

- Success! Both user and root are pwned. 