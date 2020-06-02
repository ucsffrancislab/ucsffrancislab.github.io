
#	MetaGO


MetaGO kmer analysis processing.

<https://www.frontiersin.org/articles/10.3389/fmicb.2018.00872/full>

<https://github.com/VVsmileyx/MetaGO>

I have forked the original repository as I anticipate a number of modifications.

<https://github.com/ucsffrancislab/MetaGO>



WARNING: In its current state, MetaGO parses filenames with dots and assumes that the 3rd position flags as gzipped and the second position for filetype. If, as I normally do, have other dots in the filename, you will get no results. Just Filename_without_dots.fastq.gz



Our cluster is old and won't run spark or much of our software so we'll be using the cloud.
Because of this, we'll need to move our data.



I would recommend renaming all files with Case/Control prefixes, removing any dots and shortening the names.
I just created a separate directory and linked the files.
Your call though.

```BASH
tail -n +2 DATADIR/metadata.csv | \
	awk -F, '{print "ln -s DATADIR/"$1".hg38.bowtie2-e2e.unmapped.fasta.gz ./"$2"-"$1"-unmapped.fasta.gz"}' \
	| bash

ll Control-* | wc -l
29
ll Case-* | wc -l
30

aws s3 sync ~/MetaGO_S3_DATADIR/ s3://herv-unr/MetaGO_S3_DATADIR/
```






We could do this a number of ways, but I'm gonna prepare a docker image locally.
This docker file, `MetaGO_demo.Dockerfile`, has everything that I will need to run the demo and my data.

```
FROM ubuntu:latest
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get -y update ; apt-get -y full-upgrade ; \
	apt-get -y install git python software-properties-common default-jdk wget curl htop ; \
	apt-get -y autoremove

#	python-pip for python hard to find
RUN curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"; python get-pip.py

RUN cd /root/ ; \
	wget http://apache.mirrors.hoobly.com/spark/spark-2.4.5/spark-2.4.5-bin-hadoop2.7.tgz ; \
	tar xfvz spark-2.4.5-bin-hadoop2.7.tgz ; \
	/bin/rm -rf spark-2.4.5-bin-hadoop2.7.tgz
	
RUN pip install --upgrade awscli pip numpy scipy boto3 pyspark sklearn

#	FYI: ~ / $HOME is /root

RUN mkdir ~/github/
RUN mkdir -p ~/.local/bin

RUN cd ~/github/; \
	git clone https://github.com/ucsffrancislab/MetaGO.git ; \
	cd MetaGO ; tar xfvz MetaGO_SourceCode.tar.gz

RUN cd ~/github/; \
	git clone https://github.com/VVsmileyx/TestData.git ; \
	cd TestData ; unzip testDATA.zip

RUN cd ~/github/; \
	git clone https://github.com/VVsmileyx/Tools.git ; \
	cd Tools ; tar xfvz dsk-1.6066.tar.gz ; \
	cp /root/github/Tools/dsk-1.6066/dsk ~/.local/bin/ ; \
	cp /root/github/Tools/dsk-1.6066/parse_results ~/.local/bin/

ENV PATH="/root/.local/bin:/root/github/MetaGO/MetaGO_SourceCode:${PATH}"

RUN ls -1 /root/github/TestData/testDATA/H*.fasta  > /root/github/MetaGO/MetaGO_SourceCode/fileList.txt
RUN ls -1 /root/github/TestData/testDATA/P*.fasta >> /root/github/MetaGO/MetaGO_SourceCode/fileList.txt

RUN mkdir /root/MetaGO_Result
WORKDIR /root/github/MetaGO/MetaGO_SourceCode/
```


build it and see.

```BASH
docker build -t metago --file MetaGO_demo.Dockerfile ./
```

Then run it and look around.

```BASH
docker run -it metago


...


exit
```

Once the data is uploaded and I'm satisfied with my docker image, I start an instance on AWS.
I use one of my scripts to start the latest Amazon Linux on a decent resource set.


```BASH
create_ec2_instance.bash --profile gwendt --image-id ami-0323c3dd2da7fb37d \
	--instance-type i3.2xlarge --key-name ~/.aws/JakeHervUNR.pem --NOT-DRY-RUN
```

Then login and update the system and install docker.

```BASH
ssh ...
sudo yum update
sudo yum install docker htop
```

This particular instance comes with some addition ssd "ephemeral" storage.
Its fast and local, which is one reason why I chose this instance type.
It is not mounted though, which is stupid.
So we look around and then mount it.

```BASH
df -h
lsblk
sudo mkfs -t xfs /dev/nvme0n1
mkdir ~/ssd0
sudo mount /dev/nvme0n1 ~/ssd0/
sudo chown ec2-user ~/ssd0/
```

This was 1.7TB if I remember correctly.
Some other types come with multiple ssd drives that each need prepared separately if desired.
Perhaps use ssd0 as data in, ssd1 as data out and ssd2 as docker storage, or something like that.
Only 1 on this type so let's do everything there.



On larger instance types, there are multiple 1.7TB drives.
And some data sets will require more space.
Perhaps do this by creating a raid drive across them all with something like.
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/raid-config.html
```BASH
sudo yum -y install mdadm
lsblk
sudo mdadm --create --verbose /dev/md0 --level=0 --name=MY_RAID --raid-devices=4 /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
# wait until complete, but seemed immediate
sudo cat /proc/mdstat
# wait until complete, but seemed immediate
sudo mdadm --detail /dev/md0
sudo mkfs.ext4 -L MY_RAID /dev/md0
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm.conf
sudo dracut -H -f /boot/initramfs-$(uname -r).img $(uname -r)
sudo mkdir -p /mnt/raid
sudo mount LABEL=MY_RAID /mnt/raid
sudo chown ec2-user /mnt/raid/
mkdir -p /mnt/raid/MetaGO_Result
mkdir -p /mnt/raid/docker
```




Prepare the output directory.
Download the data.
Tell docker to put its stuff there too.
Most instances only come with 8GB of main storage with isn't enough for anything really.
Start docker and give its user some power, then logout and back in.


```BASH
mkdir ~/ssd0/MetaGO_Result

aws s3 sync s3://herv-unr/MetaGO_S3_DATADIR/ ~/ssd0/MetaGO_S3_DATADIR/ 

sudo bash -c 'echo { \"data-root\":\"/home/ec2-user/ssd0/\" } >> /etc/docker/daemon.json'

sudo service docker start
sudo usermod -a -G docker ec2-user
exit
ssh ...
```


Now, my docker file was part of this repo so I'm just going to download it.
You may get yours on the instance however you need to.
Then we need to build it.


```BASH
mkdir ~/tmp/
cd ~/tmp/
wget https://raw.githubusercontent.com/ucsffrancislab/genomics/master/docker/MetaGO_demo.Dockerfile
docker build -t metago --file MetaGO_demo.Dockerfile ./
```


I'm not going to stay logged in and I want this to keep running while I'm gone.
So do not use the --rm option!
Start docker container with -itd options.
The -d will kick us out initially. Hopefully, I can find a better way.
We also want to mount the local directory with it so that the input and output are available to both.

```BASH
docker run -v /home/ec2-user/:/mnt -itd metago
```

This will have printed the container id and kicked us out.
We could see it if needed. It should show a status of running or something like that.

```BASH
docker ps -a
```

Now we need to connect to container.

```BASH
docker exec -it $( docker ps -aq ) bash
```


Create filelist on docker instance so path is correct

Each line is the complete path of raw data, such as the first line is '/home/user/MetaGO/testData/H1.fasta', and the second line is '/home/user/MetaGO/testData/H2.fasta' ... Furthermore, and the samples belong to same group must arrange togther(e.g. line1 to linek of the list belongs to group 1 (health) and linek+1 to lineN belongs to group 2 (patient) ).
-N	--n1	The number of samples belong to group 1.
-M	--n2	The number of samples belong to group 2.


```BASH
ls -1 /mnt/ssd0/MetaGO_S3_DATADIR/Control* > /root/fileList.txt
ls -1 /mnt/ssd0/MetaGO_S3_DATADIR/Control* | wc -l
29
ls -1 /mnt/ssd0/MetaGO_S3_DATADIR/Case*   >> /root/fileList.txt
ls -1 /mnt/ssd0/MetaGO_S3_DATADIR/Case* | wc -l
30

nohup bash /root/github/MetaGO/MetaGO_SourceCode/MetaGO.sh --inputData RAW --fileList /root/fileList.txt \
	--n1 $( ls -1 /mnt/ssd0/MetaGO_S3_DATADIR/Control* | wc -l ) \
	--n2 $( ls -1 /mnt/ssd0/MetaGO_S3_DATADIR/Case* | wc -l ) \
	--kMer 21 --min 1 -P 16 --ASS 0.65 --WilcoxonTest 0.1 --LogicalRegress 0.5 \
	--filterFuction ASS --outputPath /mnt/ssd0/MetaGO_Result --Union --sparse --cleanUp \
	> /mnt/ssd0/MetaGO_Result/MetaGO.output.txt 2>&1 &

```

At the moment this isn't very parallel.
dsk will only use 8 processors, no matter how many are available.
And most other things are just serial.
Perhaps this is adjustable.

Keep an eye on things with `htop` or `tail -f ~/ssd0/MetaGO_Result/MetaGO.output.txt`
Eventually, upload your data.

```BASH
aws s3 sync --delete ~/ssd0/MetaGO_Result/ s3://herv-unr/MetaGO_S3_DATADIR_Results/
```


When you are finished, shut down the instance and everything will be gone.

```BASH
sudo shutdown now
```

