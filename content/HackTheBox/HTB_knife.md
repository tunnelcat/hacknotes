# Process
- First, as always, a lazy nmap scan.

![[c3791ae8b47eca793c3722015394a8d4-1.png]]

- Basic dirb did not return anything interesting.
- Basic nikto gives more information about the server...

![[178dff6f26e26d0dd2e2b3fbcd57df49-1.png]]

- The PHP information turned out to be some good stuff... There's a backdoor for this particular PHP version.

![[3f8efaa40d28d36d5946b0241de8a2fb-1.png]]

- Using the exploit gives access to a shell.

![[560dc23628e39a36e26570720d5ce59b-1.png]]

- Though, it appears that cd is not allowed for some reason...

![[da28c76d1843576b09042965aac82ce1-1.png]]

![[10ff3a65225567f7e7baaaefb78be30f-1.png]]

- User flag is located in the home folder as always

![[af578def8ee05cecaf6a6a87d9090a23-1.png]]

- For privesc, `sudo -l` turned out to be very interesting. It turns out that /usr/bin/knife can be run with sudo with no password and it will run with root permissions.

![[afb9f44d96faa470038740ed2e050ede-1.png]]

- But first, an ssh upgrade wouldn't hurt. It's probably more stable than the current webshell. (Generated ssh keys, then moving the public key over to the target machine)

![[99d2b304019f401927d82ab609c7f560-1.png]]

![[58ea206c15c1cd289724863494e633e5-1.png]]

- Running the /usr/bin/knife program, it seems that there is a very interesting function that allows us to execute ruby scripts. This is perfect for reverse shells!

![[152653fd778ffae7e727cbcb9ebbb013-1.png]]

- Copy-pasting a Ruby reverse shell from the Internet and saving it on a file called payload.rb

![[1137359c1c9536708b37099480ebef34-1.png]]

- And finally getting a reverse shell to root by executing the script and listening with netcat on Kali

![[b70d534c27ad91b8517c9335f74b685c-1.png]]

![[bc3c98e711927ad46dde54206f710f67-1.png]]

- Pwned!

![[bfca8098dd604b03c07ec622cfae7105-1.png]]

