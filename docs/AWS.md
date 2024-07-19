
#	AWS (Amazon Web Services)

Command line access and usage of UCSF managed AWS resources.

Must be connected to UCSF VPN.


To login to the web console, https://adfs.ucsf.edu/adfs/ls/idpinitiatedSignon.aspx



##	From UCSF SEC

https://ucsfonline.sharepoint.com/sites/SecureEnterpriseCloud



[Secure Enterprise Cloud User Guide](https://ucsf.app.box.com/s/9sy28v2v0fgqe3xygsj2j9szxq431bdl)

If you ask support for help and the answers in this guide, they're gonna point that out.





##	General CLI setup

Be sure to use the `awscli` package and not the `aws` package.

```
pip uninstall aws
```

`aws` is old and busted. `awscli` is the new hotness.

```
pip install aws-adfs

#	or

python3 -m pip install --user --upgrade aws-adfs awscli
```


##	CLI Login



I called my profile "ucsf" but it can be anything really. Even "default".
You can even not include the `--profile` option which is essentially the same as using "default".

If you don't use other AWS accounts on this system, don't bother specifying profile at all.

This will ask for you UCSF email and password.

It should then do a Duo Push to your phone.

```
aws-adfs login --adfs-host=adfs.ucsf.edu --profile=ucsf
2021-03-10 14:08:19,355 [authenticator authenticator.py:authenticate] [7846-MainProcess] [4669504960-MainThread] - ERROR: Cannot extract saml assertion. Re-authentication needed?
Username: XXXXXXXXXXXXXx@ucsf.edu
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

I advise extending `adfs_config.session_duration` from 3600 to 28800.

I'm a bit surprised that 'us-east-1' is the default region as I've been told to use 'us-west-2'.
I'm going to edit my `~/.aws/config` to reflect this.

`~/.aws/credentials` are private. Do not share them.





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

which should eventually result in the creation of a bucket named like `backup-1-3-r-us-west-2.sec.ucsf.edu`.

I will update this with the actual name as part of it isn't clear.

Our shiny new bucket is `francislab-backup-73-3-r-us-west-2.sec.ucsf.edu`.

Apparently we are project 73, which we all know is the perfect number.




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

And it looks like you don't even need to specify the adfs host as it is in your config files.

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


I attempted a backup of large, individual files. Sadly, the token expires in an hour and this only is able to upload about 6 files.
Perhaps I can get longer tokens?

```
aws-adfs login

for f in GM* ; do echo $f ; aws s3 cp $f s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/raw/CCLS/bam/ --sse aws:kms --sse-kms-key-id alias/managed-s3-key  ; done
```

Testing sync to do this backup of roughly 1TB.


```
aws-adfs login

aws s3 sync --sse aws:kms --sse-kms-key-id alias/managed-s3-key --exclude \* --include \*/GM_\* /francislab/data1/raw/CCLS/ s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/raw/CCLS/
```

Failed.

```
...
upload failed: francislab/data1/raw/CCLS/bam/GM_63185.recaled.bam to s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/raw/CCLS/bam/GM_63185.recaled.bam An error occurred (ExpiredToken) when calling the UploadPart operation: The provided token has expired.
upload failed: francislab/data1/raw/CCLS/bam/GM_439338.recaled.bam to s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/raw/CCLS/bam/GM_439338.recaled.bam An error occurred (ExpiredToken) when calling the UploadPart operation: The provided token has expired.
upload failed: francislab/data1/raw/CCLS/bam/GM_634370.recaled.bam to s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/raw/CCLS/bam/GM_634370.recaled.bam An error occurred (ExpiredToken) when calling the UploadPart operation: The provided token has expired.
upload failed: francislab/data1/raw/CCLS/bam/GM_983899.recaled.bam to s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/raw/CCLS/bam/GM_983899.recaled.bam An error occurred (ExpiredToken) when calling the UploadPart operation: The provided token has expired.
```

But sync is rerunnable.
I reran and refreshed my token in a different window several times.
Still failed.


The initial loop copy could have included a `aws-adfs login` command before each file to refresh the session?

Gonna try rerunning a couple times to see if it eventually finishes?
Nope. Sync seems to be uploading multiple files at the same time.
Neither finish within an hour so it dies.
Refreshing connection and uploading each individually.

I'm probably gonna need to hunt down and delete these partial uploads with my script `~/github/jakewendt/awstools/scripts/abort_all_multipart_uploads.bash`. Yep.


Increase `adfs_config.session_duration` in `~/.aws/config`









Sync buckets
```
aws s3 sync --exclude server-access-logging/* --sse aws:kms --sse-kms-key-id alias/managed-s3-key s3://francislab-nih-temp-73-3-r-us-west-2/ s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/nih-20220303-sync/
```


How big is my bucket or folder?
```
aws s3 ls s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu/nih-20220303-sync/  --recursive --human-readable --summarize
```



##	EC2


###	SSM Plugin

Ensure SSM Session manager Extension is installed. Without it, you will be unable to start a session.

https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html

https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-sessions-start.html

I install it locally.

####	Mac

```
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/mac/sessionmanager-bundle.zip" -o "sessionmanager-bundle.zip"
unzip sessionmanager-bundle.zip
./sessionmanager-bundle/install -i ~/.local/sessionmanagerplugin -b ~/.local/bin/session-manager-plugin
```


####	Linux

Not sure how to do without sudo yet

```
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/linux_64bit/session-manager-plugin.rpm" -o "session-manager-plugin.rpm"
sudo yum install -y session-manager-plugin.rpm
```


####	Uninstall

```
sudo rm -rf /usr/local/sessionmanagerplugin /usr/local/bin/session-manager-plugin
/bin/rm -rf .local/sessionmanagerplugin/ .local/bin/session-manager-plugin

sudo yum erase session-manager-plugin -y
```


Using a session manager is meant to make things easier to control the ssh session.

https://medium.com/@dnorth98/hello-aws-session-manager-farewell-ssh-7fdfa4134696

https://www.nclouds.com/blog/ssh-session-manager/

https://cloudonaut.io/goodbye-ssh-use-aws-session-manager-instead/



###	Start, Access, Use and Terminate an instance


How to start, access and terminate EC2 instances...

I believe that you can only use UCSF blessed AMI's.

Their id is `013463732445`.

Some image names include "_dl_" which I think is for "Deep Learning".


```
aws-adfs login --adfs-host=adfs.ucsf.edu --profile=ucsf
```

As I only use this one account on C4, profile is unnecessary.

Specifying the adfs host is require the first time as it adds to your `~/.aws/config` and `~/.aws/credentials`.
```
aws-adfs login --adfs-host=adfs.ucsf.edu
```

You will likely need to edit your `~/.aws/config` however.

For some reason, the region set varies. I think that it needs to be `us-west-2`.
Mine was set to `eu-central-1`. I'm gonna leave it and see.
If you don't change the region to `us-west-2` you'll get a lot of errors like ...

```
An error occurred (UnauthorizedOperation) when calling the DescribeImages operation: You are not authorized to perform this operation.

An error occurred (UnauthorizedOperation) when calling the DescribeSubnets operation: You are not authorized to perform this operation.

An error occurred (UnauthorizedOperation) when calling the DescribeSecurityGroups operation: You are not authorized to perform this operation.
```

You will also want to up the session duration to 28800.


```
aws-adfs login

ami_id=$( aws ec2 describe-images --owners 013463732445  | jq -r '.Images | map(select(.Name | test("^base-ubuntu-18-ami"))) | sort_by(.CreationDate)[].ImageId' | tail -1 )
echo $ami_id

subnet_id=$( aws ec2 describe-subnets | jq -r '.Subnets | sort_by(.AvailableIpAddressCount) | reverse[0].SubnetId' )
echo ${subnet_id}

ssm_security_group_id=$( aws ec2 describe-security-groups | jq -r '.SecurityGroups | map(select( .GroupName == "managed-ssm" ))[].GroupId' )
echo $ssm_security_group_id

mns_security_group_id=$( aws ec2 describe-security-groups | jq -r '.SecurityGroups | map(select( .GroupName == "managed-network-services" ))[].GroupId' )
echo $mns_security_group_id

security_ids="${ssm_security_group_id} ${mns_security_group_id}"
echo $security_ids

instance_id=$( aws ec2 run-instances --image-id ${ami_id} --instance-type c5.large --subnet-id ${subnet_id} --security-group-ids ${security_ids} --iam-instance-profile Name=managed-service-ec2-standard  | jq -r '.Instances[].InstanceId' )
echo ${instance_id}


#	instance_id=$( aws ec2 run-instances --image-id ${ami_id} --instance-type c5.large --subnet-id ${subnet_id} --security-group-ids ${security_ids} --iam-instance-profile Name=managed-service-ec2-standard --block-device-mappings "DeviceName=/dev/nvme,Ebs={VolumeSize=2000,VolumeType=gp2}" | jq -r '.Instances[].InstanceId' )

#		"--block-device-mappings DeviceName=/dev/xvda,Ebs={VolumeSize=2000,VolumeType=gp2}"
#	64TB maximum?
#	https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/volume_constraints.html

#	pass xvda by it uses nvme1n1 anyway.

#	Tried my name, no name. Name is required, but then ignored by NVMe?

#	seems to use its own naming convention.
#	$ lsblk
#	NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
#	loop1         7:1    0 55.5M  1 loop /snap/core18/2284
#	loop2         7:2    0 25.1M  1 loop /snap/amazon-ssm-agent/5429
#	loop3         7:3    0 43.6M  1 loop /snap/snapd/14978
#	loop4         7:4    0 25.1M  1 loop /snap/amazon-ssm-agent/5521
#	nvme1n1     259:0    0  1.5T  0 disk                             <---
#	nvme0n1     259:1    0   40G  0 disk
#	└─nvme0n1p1 259:2    0   40G  0 part /

#	> tmp_run_instances ???
# Really should catch an identifier here as the next command is not specific to this command.
#	then pipe through jq to grab the instance id or equivalent
#	instance id is in there





#--tag-specifications 'ResourceType=instance,Tags=[{Key="Patch Group",Value="ubuntu-prod"}]' --metadata-options 'HttpTokens=required,HttpEndpoint=enabled' --dry-run

#	These two things actually happen automatically on creation.

#Metadata version
#V2 (token required)
#For V2 requests, you must include a session token in all instance metadata requests. Applications or agents that use V1 for instance metadata access will break.
#	This may be the default setting.
#	HttpTokens
#	aws ec2 modify-instance-metadata-options  --http-endpoint enabled --http-token required
#	 --metadata-options 'HttpTokens=required,HttpEndpoint=enabled'
#	aws ec2 modify-instance-metadata-options --instance-id <INSTANCE-ID> --profile <AWS_PROFILE> --http-endpoint enabled --http-token required
#
#Click “Next: Add Tags” to proceed with resource tagging14.IMPORTANT: Add a tag with key = “Patch Group” andvalue =<os>-proda.For Windows instances: value = “windows-prod”b.For Ubuntu instances: value = “ubuntu-prod”c.For Centos instances: value = “centos-prod”d.Note:Clientsareresponsibleforensuringthe“PatchGroup”tagisappliedtoallinstanceslaunchedwithintheiraccount.Thistagisofcriticalimportance,andtriggersSECautomatedpatchingandmanagement.Failuretoapply an appropriate Patch Group tag may result in the termination of your instance!
#
#	this my be the default as well




#	... WAIT A MINUTE OR TWO ...

#	if you didn't catch the instance id, you can search for it ...
#instance_id=$( aws ec2 describe-instances | jq -r '.Reservations[].Instances | map(select( .State.Name == "running"))[].InstanceId' )
#echo ${instance_id}


#	... WAIT ANOTHER MINUTE OR TWO ...


aws ssm start-session --target ${instance_id}

Starting session with SessionId: XXXXXXXXXX@ucsf.edu-.......
This session is encrypted using AWS KMS.






#	So connected, but I quickly found that there is still much to do.

#	These images aren't really set up well.
#	Home dir isn't even owned by my user.

whoami
# ssm-user

sudo chown ssm-user $HOME
cd
bash

alias ll="ls -l"

#	Mount unmounted drives

#	If added a volume ... (perhaps use xfs instead of ext4)
lsblk
sudo file -s /dev/nvme1n1
sudo mkfs -t ext4 /dev/nvme1n1
sudo file -s /dev/nvme1n1
sudo mkdir /data
sudo mount /dev/nvme1n1 /data
sudo chmod 777 /data






#	Looks like apt update runs automatically and can't run 2 at the same time.
#	There may be collisions. Try rerunning.
#	Or maybe I'm not supposed to run it at all?
#	The user guide does use it to install a couple things.

sudo apt-get clean all
sudo apt-get -y update
sudo apt-get -y upgrade
sudo apt-get -y autoremove


aws s3 ls
2021-03-11 08:01:44 francislab-backup-73-3-r-us-west-2.sec.ucsf.edu
2021-02-19 15:24:38 managed-755550924152-server-access-logs

aws s3 ls s3://francislab-backup-73-3-r-us-west-2.sec.ucsf.edu

#	working now



#sudo apt-get -y install python3-setuptools bash python3 git wget gcc g++ make bzip2 libbz2-dev zlib1g-dev libncurses5-dev libncursesw5-dev liblzma-dev libcurl4-openssl-dev

#python3 -m pip install --upgrade --user pip

#python3 -m pip install --upgrade --user aws-adfs



#	Do stuff




exit

aws ec2 terminate-instances --instance-ids ${instance_id}

aws ec2 describe-instances | jq -r '.Reservations[].Instances[].State.Name'
```







##	EBS


On an instance, if you didn't request enough storage initially, you can add storage like EBS.

You should be able to describe, create, attach, mount, unmount, detach, destroy volumes.

```
sudo apt-get install -y jq

TOKEN=$( curl -sX PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
instance_id=$( curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/instance-id 2> /dev/null )
echo $instance_id
az=$( curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/placement/availability-zone 2> /dev/null )
echo $az
region=$( curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/meta-data/placement/region 2> /dev/null )
echo $region


# in GB (what is max?)
ebs_size=100



command="aws ec2 describe-volumes --region ${region}"
echo $command
response=$( $command )
echo "$response"


command="aws ec2 create-volume --encrypted --kms-key-id alias/aws/ebs --region ${region} --availability-zone ${az} --volume-type gp2 --size ${ebs_size}"
echo $command
response=$( $command )
echo "$response"
volume_id=$( echo $response | jq -r '.VolumeId' )
echo ${volume_id}


#	I still don't get device names. How do I locate this new volume?
#	There is no sdf
aws ec2 attach-volume --region ${region} --device /dev/sdf --instance-id ${instance_id} --volume-id ${volume_id}

{
    "AttachTime": "2022-04-13T15:43:48.784Z",
    "Device": "/dev/sdf",
    "InstanceId": "i-0a4b3df7aa0e14095",
    "State": "attaching",
    "VolumeId": "vol-06181d4fb21c119e9"
}
ssm-user@ip-10-90-179-177:~$ lsblk
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop1         7:1    0 43.6M  1 loop /snap/snapd/14978
loop2         7:2    0 25.1M  1 loop /snap/amazon-ssm-agent/5656
loop3         7:3    0 55.5M  1 loop /snap/core18/2284
loop4         7:4    0 25.1M  1 loop /snap/amazon-ssm-agent/5688
loop5         7:5    0 43.6M  1 loop /snap/snapd/15177
loop6         7:6    0 55.5M  1 loop /snap/core18/2344
nvme0n1     259:0    0   40G  0 disk 
└─nvme0n1p1 259:1    0   40G  0 part /
nvme1n1     259:2    0  100G  0 disk 

#	Not helpful


#	Think this would work programmatically
new_volume=$( ls -dltr /dev/ | grep disk | tail -n 1 | awk '{print $NF}' )
echo ${new_volume}

#/dev/nvme1n1



sudo file -s /dev/nvme1n1
sudo mkfs -t ext4 /dev/nvme1n1
sudo file -s /dev/nvme1n1
sudo mkdir /data
sudo mount /dev/nvme1n1 /data
sudo chmod 777 /data




aws ec2 detach-volume --region ${region} --volume-id ${volume_id}

aws ec2 delete-volume --region ${region} --volume-id ${volume_id}

```




