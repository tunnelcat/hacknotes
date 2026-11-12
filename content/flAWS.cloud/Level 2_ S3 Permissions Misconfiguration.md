First, create a free-tier AWS account to use the AWS CLI. 
Create an Administrator user in IAM and generate an access key and secret key (save this for later). Follow the AWS documentation for the most up-to-date process. 

Install `awscli` and configure with `aws configure` (need your access and secret key)
```
sudo apt install awscli
aws configure
```

![94efc742b5aaadba7f5dc48769ce96ce.png](../../_resources/94efc742b5aaadba7f5dc48769ce96ce-1.png)

Now we can see what we can do with our AWS permissions. For example, trying to list the S3 bucket files: 
```
aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud 
```

![26882ae238d6babb1b568a2924a41ab0.png](../../_resources/26882ae238d6babb1b568a2924a41ab0-1.png)

Success! The flag is at `http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/secret-e4443fc.html`

![9e5a298dfb69bbff8c5374af711b5362.png](../../_resources/9e5a298dfb69bbff8c5374af711b5362-1.png)

Lesson: Similar to opening permissions to "Everyone", people accidentally open permissions to "Any Authenticated AWS User". They might mistakenly think this will only be users of their account, when in fact it means anyone that has an AWS account. 