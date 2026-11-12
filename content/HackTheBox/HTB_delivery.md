# Takeaways
- Sometimes, the answer is right in front of you.
	- Understand what the machine is doing on a bigger picture. What is the purpose? 
	- Try trivial credentials (and keywords from the machine)
	- If you get credentials, try them out on everything (even seemingly dumb SSH)
- Enumeration is essential. There is no shortcut, but you can enumerate smart.
	- Look at config files/folders
	- Look at folders with non-standard software

# Process
- First, lazy nmap scan as usual...

![cece3482da588d3e653a3d42e8e778e0.png](../../_resources/cece3482da588d3e653a3d42e8e778e0-1.png)

- Going to the webserver on port 80 reveals a website with a contact us page with some interesting links.

![ec26e065a675a6d9e761addedb7fa838.png](../../_resources/ec26e065a675a6d9e761addedb7fa838-1.png)

![513865ebd4cee02c014235c056463b66.png](../../_resources/513865ebd4cee02c014235c056463b66-1.png)

- Now we "poison" our own /etc/hosts file to resolve the URLs to the correct IP address.

![2fbd8c71d2ab89f96db1de1457bf7adf.png](../../_resources/2fbd8c71d2ab89f96db1de1457bf7adf-1.png)

- We see three "sites" in total:

![b366833d2f67527be4001c8773a48987.png](../../_resources/b366833d2f67527be4001c8773a48987-1.png)

![3282e75bb24edb2ccde4d4cc4414f2e6.png](../../_resources/3282e75bb24edb2ccde4d4cc4414f2e6-1.png)

![3386f13c5a488b9f7de4899a98c951ef.png](../../_resources/3386f13c5a488b9f7de4899a98c951ef-1.png)

- owo what's this? 

![9ccceb783ef3df5dd9999a414760cc3e.png](../../_resources/9ccceb783ef3df5dd9999a414760cc3e-1.png)

![6602281412d9b77a88a3c9a0dcb8c4bd.png](../../_resources/6602281412d9b77a88a3c9a0dcb8c4bd-1.png)

- Is there an arbitrary file upload on the osticket (helpdesk.deliver.htb) webapp?

![1d5505447093b92810e7f9eb50ada42d.png](../../_resources/1d5505447093b92810e7f9eb50ada42d-1.png)

![2b7ab6c2e4112291561f12be291af4f9.png](../../_resources/2b7ab6c2e4112291561f12be291af4f9-1.png)

### Getting a delivery.htb email

- First, we create a ticket and view it, taking note of the email and ticket number.

![6c6e52d2e4ce0193c2351e1d0fb3cf13.png](../../_resources/6c6e52d2e4ce0193c2351e1d0fb3cf13-1.png)

- Note that emailing to [ticket number]@delivery.htb adds it to our ticket. Interesting...

![e2bf24d693e5b6f08aac0ee529dfc3fc.png](../../_resources/e2bf24d693e5b6f08aac0ee529dfc3fc-1.png)

- Making a user with "password" as password. Notice the ticket number email. The site will email us a confirmation to the email address, which will add it to the ticket!

![d4a23662a9741f4450be834de019a485.png](../../_resources/d4a23662a9741f4450be834de019a485-1.png)

- Now we check the ticket status created earlier. Nothing? Ok, then...

![a0825a17fd4682dedbdba85cc18ede69.png](../../_resources/a0825a17fd4682dedbdba85cc18ede69-1.png)

- Trying Mattermost user sign-up... user:Password_123

![5c24800adaa08dd7a1a4e9dfacb07a35.png](../../_resources/5c24800adaa08dd7a1a4e9dfacb07a35-1.png)

![54595aeb89c2cd1484fdfac166ef70ba.png](../../_resources/54595aeb89c2cd1484fdfac166ef70ba-1.png)

- Check back on the ticket, and ...? Success! An account is now registered for the Mattermost server! (We can activate it using the link)

![d68a6be6f2f96219a962c5cce2613418.png](../../_resources/d68a6be6f2f96219a962c5cce2613418-1.png)

- After logging in to Mattermost, we get some interesting information in the Internal channel.
	- hashcat password cracking hint
	- osTicket agent credentials

![8506b070bbf56ef20e81e4cf59093bfb.png](../../_resources/8506b070bbf56ef20e81e4cf59093bfb-1.png)

![e989244cd2723efa67f7a072e0c654ea.png](../../_resources/e989244cd2723efa67f7a072e0c654ea-1.png)

- So the credentials as an agent are maildeliverer:Youve_G0t_Mail!
- Also, we get a nice hint about hashcat and cracking passwords for later

![3d2a4d97cd61cf10be6b1431eedb0007.png](../../_resources/3d2a4d97cd61cf10be6b1431eedb0007-1.png)

- Looking in the Admin panel, we can see that the osTicket version is v1.15.1. This helps narrow down the CVE exploits that are applicable, though it seems that there are none that are applicable.

- Looking at the users tab, there seems to be some suspicious hash-looking items... Maybe this is what should be used for hashcat.

![cc175aec93bcc00c96d800d16cf30dcb.png](../../_resources/cc175aec93bcc00c96d800d16cf30dcb-1.png)

- These are the 4 user hashes: 
	- 5b785171bfb34762a933e127630c4860
	- 9ecfb4be145d47fda0724f697f35ffaf
	- c3ecacacc7b94f909d04dbfd308a9b93
	- ff0a21fc6fc2488195e16ea854c963ee

- They look like MD5 hashes? 32-chars and using hash-identifier
- Seems like this may be a rabbit hole... I tried using rule-based attacks on hashcat to no avail. Oh well. 
- People are generally lazy. Let's see if the maildeliverer:Youve_G0t_Mail! credentials are used somewhere else... (like port 22, SSH)

- ... I- Okay.

![e19c8b67d2266fa14f5a7ca807ddaace.png](../../_resources/e19c8b67d2266fa14f5a7ca807ddaace-1.png)

![d41dc503056be8b3af38f963efd86d03.png](../../_resources/d41dc503056be8b3af38f963efd86d03-1.png)

- After a LOT of browsing and some Googling, I found some mysql credentials in the osticket folder: /var/www/osticket/upload/include/ost-config.php

![7dcc1bff2bb997872bb643f917c1b6a1.png](../../_resources/7dcc1bff2bb997872bb643f917c1b6a1-1.png)

![e9ffcf2c908c5083db6f90c819e8eb3c.png](../../_resources/e9ffcf2c908c5083db6f90c819e8eb3c-1.png)

- After more browsing, it turns out that there is an interesting bit in: /opt/mattermost/config/config.json

![a95f56f55b29f727c3bf36e3471d44f2.png](../../_resources/a95f56f55b29f727c3bf36e3471d44f2-1.png)

- There are plaintext credentials to a MySQL server...

```
mmuser:Crack_The_MM_Admin_PW@tcp(127.0.0.1:3306)/mattermost?charset=utf8mb4,utf8\u0026readTimeout=30s\u0026writeTimeout=30s"
```

- Login with the following command: 
```
mysql -h 127.0.0.1 -P 3306 --user=mmuser --password=Crack_The_MM_Admin_PW
```

- Then, list the available databases that the user can access with `SHOW DATABASES;`

![01197b184b84a2d9d9a2570d6d739672.png](../../_resources/01197b184b84a2d9d9a2570d6d739672-1.png)

- Set to use the mattermost database with `use mattermost;`

- And, after listing the tables with `show tables;`, the Users table stands out (it might have credentials), so get some details about the structure of the table with `describe Users;`

![89af9058b13b32b94eb5187020910f16.png](../../_resources/89af9058b13b32b94eb5187020910f16-1.png)

- In particular, the Username and Password columns are interesting, so list those out with `select Username,Password from Users;`

![3f18dc1eb88c8a8ca26785de12dfcf9a.png](../../_resources/3f18dc1eb88c8a8ca26785de12dfcf9a-1.png)

- Copy-pasting the root password hash into a text file called `target_hashes.txt` should work for this small set of hashes.
- Note that the hash type is bcrypt, after looking around on Google and Wikipedia.
- Now, fire up hashcat. Remember that we are cracking bcrypt hashes and trying out variations of the "PleaseSubscribe!" hint we got earlier. One way to generate variations in hashcat is called a "rule-based attack"
- Many `.rule` files for hashcat are available online via Google, but I like `OneRuleToRuleThemAll.rule`
- Note that the `wordlist.txt` contains a single entry with "PleaseSubscribe!"
- Run hashcat and pray!

```sh
hashcat -m 3200 target_hashes.txt wordlist.txt -r OneRuleToRuleThemAll.rule
```

- After some time, the cracked password is revealed:

![6ad181fe1baee41d9a1971363e01f5a8.png](../../_resources/6ad181fe1baee41d9a1971363e01f5a8-1.png)

- And after switching user to root with the password, we get the flag for root.

![c90abcea075cd6da262b8445503edc53.png](../../_resources/c90abcea075cd6da262b8445503edc53-1.png)

- Done!