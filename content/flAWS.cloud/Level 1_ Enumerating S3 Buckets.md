It turns out that `flaws.cloud` is a site hosted using AWS S3 buckets.

A simple `dig` and `nslookup` of the IP can yield interesting results: 
```
dig flaws.cloud
nslookup <ip from dig result>
```

![[540470bea389d8ee672eaab7f78d39de-1.png]]

Therefore, an alternative URL for `flaws.cloud` is `http://flaws.cloud.s3-website-us-west-2.amazonaws.com/`

An `amass` scan reveals multiple interesting subdomains, but this is coincidental and not really in scope for this level. 3
```
amass enum -d flaws.cloud
```

![[f87df05fa4d1eedea15aa34a0bdcbe8b-1.png]]

Putting these subdomains into a txt file called `bucket-names.txt` and using `s3scanner` to further enumerate the buckets reveals that the `flaws.cloud` bucket has a global allow read permission. 
```
s3scanner --threads 8 scan --buckets-file ./bucket-names.txt    
```

![[f0faa2bd860a4a3fecda689d28361c38-1.png]]

Since S3 buckets are usually reachable via the format `http://<bucketname>.s3.amazonaws.com` or `http://s3.amazonaws.com/<bucketname>`, we can use techniques like Google Dorking to find potentially interesting leaks.

Google search query:
```
site:s3.amazonaws.com flaws.cloud
```

![[78a1386f477b576b2d330eee23106268-1.png]]

We can see that there is a secret html file hidden. 

![[fe483417102acb46484129f8f9feba79-1.png]]

Visiting `http://flaws.cloud/secret-dd02c7c.html` reveals the flag for Level 1!

![[1af2427b6c194342e92328bcc46b30f9-1.png]]

Lesson: Just like how you shouldn't allow directory listings of web servers, you shouldn't allow bucket listings. 