## How to test for info disclosure
- Don't get tunnel vision. There are many vectors for info disclosure
- Key skill: recognize interesting info
	- Fuzzing
	- Burp Scanner
	- Burp engagement tools - right click HTTP msg, "Engagement tools"
		- Search for diffs
		- Find comments
		- etc.
	- Engineering informative responses (submit invalid to cause response error msgs that disclose specific info)

## Files for web crawlers
- Includes `robots.txt` and `sitemap.xml`

## Directory listings
- Might automatically list contents of directories without index pages...

## Dev comments
- In-line HTML comments? might be a hint

![[c40e23d64b082eaa528d9ca5d4d3bc65-1.png]]

## Error messages
- Pay attention to any error messages during audits
- May reveal versions and technologies (public exploits?)
- Using open-source frameworks? (can audit code externally)

![[eca340eac203738e1d14ebdaac61ec6b-1.png]]

## Debugging data
- May find secrets in debugging data or pages
	- keys
	- hostnames/credentials
	- file/directory names

![[435d77676407f49588db102ad70ebf44-1.png]]

## User account pages
- Profiles usu. have sensitive info (phone numbers, API keys, etc)
- Maybe there is a logic flaw to access profiles without proper authentication
- Maybe IDOR vulns? 

## Source code disclosure (backup files)
- There may be API keys and creds in backup files. Maybe hidden somewhere in an unknown directory, so check robots.txt and brute-force lists and such

## Misconfiguration
- Sometimes, there are hidden custom HTTP headers (may allow auth bypass). To see these, HTTP TRACE method needs to be enabled. Just replace GET with TRACE, and you may find interesting headers that can be used to GET or POST other resources

![[411476c55e0ffdbd1c949bf8d0d292b3-1.png]]

![[f86ebe55fddf7f50209f3209b2468ca7-1.png]]


## Version control history
- By default, Git projects store version control data in the `.git` folder. If this folder is exposed, you might be able to get previous commits or other interesting info using tools like **GitTools** that can download and extract the "hidden" data.


