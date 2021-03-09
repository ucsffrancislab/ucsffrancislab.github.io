
#	AWS

Command line access and usage of UCSF managed AWS resources.

https://ucsfonline.sharepoint.com/sites/SecureEnterpriseCloud


##	EC2

How to start, access and terminate EC2 instances...











##	S3

###	Create New Bucket

Creating a new S3 bucket requires an email to Cloud.Support@ucsf.edu containing something like ...

```
Please create an S3 bucket for our new AWS account.
 
Account ID = ....
 
Tag = Backup
 
Data Classification = P3
 
Control Point = R
 
Description = General secure offsite storage and backup
```

which should eventually result in the creation of a bucket named `backup-1-3-r-us-west-2.sec.ucsf.edu`.

I will update this with the actual name as part of it isn't clear.




###	Read to and write from bucket



```

aws s3 ls ...

```





