
#	AWS (Amazon Web Services)

Command line access and usage of UCSF managed AWS resources.

Must be connected to UCSF VPN.


To login to the web console, https://adfs.ucsf.edu/adfs/ls/idpinitiatedSignon.aspx



##	From UCSF SEC

https://ucsfonline.sharepoint.com/sites/SecureEnterpriseCloud


 



##	General CLI setup

Be sure to use the `awscli` package and not the `aws` package.

```
pip uninstall aws
```

`aws` is quite old and busted. `awscli` is the new hotness.

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


###	SSM Plugin

Ensure SSM Session manager Extension is installed. Without it, you will be unable to start a session.

https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html

I install it locally.

####	Mac

```BASH
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/mac/sessionmanager-bundle.zip" -o "sessionmanager-bundle.zip"
unzip sessionmanager-bundle.zip
./sessionmanager-bundle/install -i ~/.local/sessionmanagerplugin -b ~/.local/bin/session-manager-plugin
```


####	Linux

Not sure how to do without sudo yet

```BASH
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/linux_64bit/session-manager-plugin.rpm" -o "session-manager-plugin.rpm"
sudo yum install -y session-manager-plugin.rpm
```


####	Uninstall

```BASH
sudo rm -rf /usr/local/sessionmanagerplugin /usr/local/bin/session-manager-plugin
/bin/rm -rf .local/sessionmanagerplugin/ .local/bin/session-manager-plugin 

sudo yum erase session-manager-plugin -y
```


###	Start, Access, Use and Terminate an instance


How to start, access and terminate EC2 instances...

I believe that you can only use UCSF blessed AMI's.

Their id is `013463732445`.


```BASH
aws ec2 describe-images --owners 013463732445

ami_id=$( aws ec2 describe-images --owners 013463732445  | jq -r '.Images | map(select(.Name | test("^base-ubuntu-18"))) | sort_by(.CreationDate)[].ImageId' | tail -1 )
echo $ami_id

subnet_id=$( aws ec2 describe-subnets | jq -r '.Subnets | sort_by(.AvailableIpAddressCount) | reverse[0].SubnetId' )
echo ${subnet_id}

security_group_id=$( aws ec2 describe-security-groups | jq -r '.SecurityGroups | map(select( .GroupName == "managed-ssm" ))[].GroupId' )
echo $security_group_id



aws ec2 run-instances --image-id ${ami_id} --instance-type t3.small --subnet-id ${subnet_id} --security-group-ids=${security_group_id} --iam-instance-profile Name='managed-service-ec2-standard'



#	metadata version - v2
#	possibly
#	--metadata-options HttpTokens=required
#	Seems starts out optional then is set to required after a few seconds.
#            "MetadataOptions": {
#                "State": "pending",
#                "HttpTokens": "optional",
#                "HttpPutResponseHopLimit": 1,
#                "HttpEndpoint": "enabled"
#            },
#
#
#                    "MetadataOptions": {
#                        "State": "applied",
#                        "HttpTokens": "required",
#                        "HttpPutResponseHopLimit": 1,
#                        "HttpEndpoint": "enabled"
#                    },





instance_id=$( aws ec2 describe-instances | jq -r '.Reservations[].Instances | map(select( .State.Name == "running"))[].InstanceId' )
echo ${instance_id}


#	... WAIT A MINUTE OR TWO ...


aws ssm start-session --target ${instance_id}

Starting session with SessionId: George.Wendt@ucsf.edu-0033bfcb90977e788
This session is encrypted using AWS KMS.




#	What user am I?
#	Do I have sudo access?




aws ec2 terminate-instances --instance-ids ${instance_id}

aws ec2 describe-instances | jq -r '.Reservations[].Instances[].State.Name'
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

2021-03-11 09:01:47 francislab-backup-73-3-r-us-west-2.sec.ucsf.edu
2021-02-19 16:24:41 managed-755550924152-server-access-logs

```

This will set (and overwrite) values in `~/.aws/credentials` and `~/.aws/config` for this profile.

If you choose to use "default" as your profile, you may never need to specify it at all.
You can even not include the `--profile` option which is essentially the same as using "default".

```
aws-adfs login --adfs-host=adfs.ucsf.edu
...

aws s3 ls

2021-03-11 09:01:47 francislab-backup-73-3-r-us-west-2.sec.ucsf.edu
2021-02-19 16:24:41 managed-755550924152-server-access-logs
```


###	Uploading Files to S3 

In order to upload files to S3 from the CLI they must be explicitly encrypted with a KMS key.  

The kms key is apparently part of the account so just add the `--sse aws:kms --sse-kms-key-id alias/managed-s3-key` options to the `cp` command.


```
aws s3 cp test.sh s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/ --sse aws:kms --sse-kms-key-id alias/managed-s3-key 

aws s3 rm s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/test.sh
```




