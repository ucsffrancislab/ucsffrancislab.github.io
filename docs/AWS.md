
#	AWS (Amazon Web Services)

Command line access and usage of UCSF managed AWS resources.

Must be connected to UCSF VPN.


To login to the web console, https://adfs.ucsf.edu/adfs/ls/idpinitiatedSignon.aspx



##	From UCSF SEC

https://ucsfonline.sharepoint.com/sites/SecureEnterpriseCloud

Ensure SSM Session manager Extension is installed 
No linux version to use from cluster. Needed?



Once approved, select the account and role you want the profile to be associated with. Enter the number corresponding to the account and role:  ????

 

Set the AWS_PROFILE environment variable  to the desired AWS Profile to authenticate with your account: “export AWS_PROFILE=<aws-profile-name>” 	#	Why?

 






##	General CLI setup

Be sure to use the `awscli` package and not the `aws` package.

```
pip uninstall aws
```


```
pip install aws-adfs

#	or

python3 -m pip install --user --upgrade aws-adfs awscli
```


Using a session manager is meant to make things easier to control the ssh session.

https://medium.com/@dnorth98/hello-aws-session-manager-farewell-ssh-7fdfa4134696

https://www.nclouds.com/blog/ssh-session-manager/

https://cloudonaut.io/goodbye-ssh-use-aws-session-manager-instead/


We shall see.


##	CLI Login



I called my profile "ucsf" but it can be anything really. Even "default".
You can even not include the `--profile` option which is essentially the same as using "default".

This will ask for you UCSF email and password.

It should then do a Duo Push to your phone.

```
aws-adfs login --adfs-host=adfs.ucsf.edu --profile=ucsf
2021-03-10 14:08:19,355 [authenticator authenticator.py:authenticate] [7846-MainProcess] [4669504960-MainThread] - ERROR: Cannot extract saml assertion. Re-authentication needed?
Username: George.Wendt@ucsf.edu
Password: 
Sending request for authentication
Waiting for additional authentication
Triggering authentication method: 'Duo Push'
Going for aws roles

        Prepared ADFS configuration as follows:
            * AWS CLI profile                   : 'ucsf'
            * AWS region                        : 'us-east-1'
            * Output format                     : 'json'
            * SSL verification of ADFS Server   : 'ENABLED'
            * Selected role_arn                 : 'arn:aws:iam::755550924152:role/managed-saml-projectuser'
            * ADFS Server                       : 'adfs.ucsf.edu'
            * ADFS Session Duration in seconds  : '28800'
            * Provider ID                       : 'urn:amazon:webservices'
            * S3 Signature Version              : 'None'
            * STS Session Duration in seconds   : '3600'
            * SSPI:                             : 'False'
            * U2F and default method            : 'True'

```

This will set (and overwrite) values in `~/.aws/credentials` and `~/.aws/config` for this profile.

You do not need to login before every command, but it does need refreshing and will eventually expire.

I'm a bit surprised that 'us-east-1' is the default region as I've been told to use 'us-west-2'.
I'm going to edit my `~/.aws/config` to reflect this.

`~/.aws/credentials` are private. Do not share them.




##	EC2

How to start, access and terminate EC2 instances...


```BASH
aws ec2 describe-images --owners 013463732445

subnet_id=$( aws ec2 describe-subnets | jq '.Subnets | sort_by(.AvailableIpAddressCount) | reverse[0].SubnetId' | tr -d '"' )
echo ${subnet_id}

aws ec2 run-instances --dry-run --image-id ami-02932400de2c9d16f --instance-type t2.micro --subnet-id ${subnet_id}


```







Login to an EC2 Instance 

Authenticate with UCSF ADFS using aws-adfs 

```
aws ssm start-session --target instance-id –-profile=<aws-profile> 
```

 

https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html 

 









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


I called my profile "ucsf" but it can be anything really.

```
aws-adfs login --adfs-host=adfs.ucsf.edu --profile=ucsf
...

aws --profile=ucsf s3 ls

2021-03-10 13:26:44 fransislab-backup-73-3-r-us-west-2.sec.ucsf.edu
2021-02-19 16:24:41 managed-755550924152-server-access-logs

```

This will set (and overwrite) values in `~/.aws/credentials` and `~/.aws/config` for this profile.

If you choose to use "default" as your profile, you may never need to specify it at all.
You can even not include the `--profile` option which is essentially the same as using "default".

```
aws-adfs login --adfs-host=adfs.ucsf.edu
...

aws s3 ls

2021-03-10 13:26:44 fransislab-backup-73-3-r-us-west-2.sec.ucsf.edu
2021-02-19 16:24:41 managed-755550924152-server-access-logs
```


###	Uploading Files to S3 

In order to upload files to S3 from the CLI they must be explicitly encrypted with a KMS key.  

The kms key is apparently part of the account so just add the `--sse aws:kms --sse-kms-key-id alias/managed-s3-key` options to the `cp` command.


```
aws s3 cp test.sh s3://fransislab-backup-73-3-r-us-west-2.sec.ucsf.edu/ --sse aws:kms --sse-kms-key-id alias/managed-s3-key 

aws s3 rm s3://fransislab-backup-73-3-r-us-west-2.sec.ucsf.edu/test.sh
```




