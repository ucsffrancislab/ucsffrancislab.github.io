
#	AWS

Command line access and usage of UCSF managed AWS resources.


##	From UCSF SEC

https://ucsfonline.sharepoint.com/sites/SecureEnterpriseCloud

```
pip install aws-adfs 
```

Ensure SSM Session manager Extension is installed 
No linux version to use from cluster. Needed?

```
aws-adfs login --adfs-host=adfs.ucsf.edu --profile=<aws-profile> 
```


Username: UCSF Email 

Password: UCSF Password 

 

Approve the Duo Request sent to your device 


Once approved, select the account and role you want the profile to be associated with. Enter the number corresponding to the account and role: 

 

Your ~/.aws/config and ~/.aws/creds files will be updated with the appropriate credentials to login to the selected account 


Set the AWS_PROFILE environment variable  to the desired AWS Profile to authenticate with your account: “export AWS_PROFILE=<aws-profile-name>” 	#	Why?

 
 
Common AWS CLI Login Issues 
UCSF VPN 
If you are connected to UCSF VPN, ADFS Authentication will fail with the error pictured below.  
To resolve, disconnect from VPN and complete the steps listed in AWS CLI Access 
Once authenticated, you may re-connect to VPN and use the credentials to use the AWS CLI 

Duo Push 
During authentication Duo cannot authenticate with following error: Error: Cannot begin authentication process. The error response: {"message": "Unknown authentication method.", "stat": "FAIL"} 
This is caused because Duo requires preferred authentication method to be set 
To resolve this, authenticate with ADFS in your browser https://adfs.ucsf.edu/adfs/ls/IdpInitiatedSignon.aspx 
Setup preferred authentication method in duo-security settings (settings' -> 'My Settings & Devices'). 

Uploading Files to S3 

In order to upload files to S3 from the CLI they must be explicitly encrypted with a KMS key.  

Pre-Requisites 

Must be connected to UCSF VPN 

Instructions 

Authenticate with UCSF ADFS using aws-adfs 

```
aws s3 cp /local/path/file s3://bucket-name/file --sse aws:kms --sse-kms-key-id alias/managed-s3-key  
```

 

Login to an EC2 Instance 

Pre-Requisites 

Must be connected to UCSF VPN 

Instructions 

Authenticate with UCSF ADFS using aws-adfs 

```
aws ssm start-session --target instance-id –-profile=<aws-profile> 
```

 

https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html 

 


 






##	General CLI setup

```
pip install aws-adfs
```



Using a session manager is meant to make things easier to control the ssh session.

https://medium.com/@dnorth98/hello-aws-session-manager-farewell-ssh-7fdfa4134696

https://www.nclouds.com/blog/ssh-session-manager/

https://cloudonaut.io/goodbye-ssh-use-aws-session-manager-instead/


We shall see.








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





