
#	Docker


Sadly, C4 uses Singularity and the CGC uses Docker.



##	How to cleanup everything


Even after stopping and deleting all containers and removing all images ...

```
docker rm $( docker ps -qa )

docker rmi -f $( docker images -q )
```
... there is still stuff taking up space.

```
du -sh ~/Library/Containers/com.docker.docker/Data/vms/
31G	~/Library/Containers/com.docker.docker/Data/vms/

docker system prune -a --volumes

du -sh ~/Library/Containers/com.docker.docker/Data/vms/
3.7G	/Users/jake/Library/Containers/com.docker.docker/Data/vms/
```

Still a bit left behind.




https://stackoverflow.com/questions/39878939/docker-filling-up-storage-on-macos

These three commands clear down anything not being used:

docker rm $(docker ps -f status=exited -aq) - remove stopped containers

docker rmi $(docker images -f "dangling=true" -q) - remove image layers that are not used in any images

docker volume rm $(docker volume ls -qf dangling=true) - remove volumes that are not used by any containers.




##	How to start a simple docker container?

docker run -ti ubuntu

With the -ti options, you will be placed inside the container as root in /



##	How to list existing / running containers?

```
docker ps --all
```

or

```
docker ps -a
```

##	How to connect to a running container?

View said instance (aka. container)

```
docker ps --all

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                      PORTS               NAMES
56e214e76b9e        ubuntu              "/bin/bash"         About a minute ago   Exited (0) 10 seconds ago                       thirsty_tesla
```


This works well.

```
docker exec -it 56e214e76b9e /bin/bash

```


Restart and attach to it

(`docker start --attach 56e214e76b9e`) this way can hang?

```
docker start 56e214e76b9e
docker attach 56e214e76b9e

root@56e214e76b9e:/#
```




##	How to list existing local docker images?


##	How to list remote docker images?


##	How to change the size of a docker container?


##	How to pass a file to a container while starting it?

For example, database credentials.


##	How to have a running container copy a file from itself to the calling host?

You can mount a directory on the container when starting it with the --volume (-v) option.

```
docker run -v /Users/Shared:/mnt -ti ubuntu
```

I doubt that this would work if run remotely.

##	How to create a docker image?

Start a docker container.
Make all of your mods.
Then …

```
docker commit RUNNING_CONTAINER_ID NEW_IMAGE_NAME
```

This new image will be local. You can push it to any number of hosts.


##	How to create a docker image from a Dockerfile?



In an otherwise empty directory, create a text file called Dockerfile. Not sure of the details yet, but other files are read or run which can wreak havoc.

In it, add instructions on creating your image. Then run “docker build -t”. Many options are available.

Be advised, apparently, after each statement an image, or the like, is created so make sure each statement cleans up after itself in the same statement. Otherwise, large files can make the final image rather large, unnecessarily so.


For example ...

```
FROM ubuntu

#ENV HOME /root
#	stop complaints like 
#	debconf: (Can't locate Term/ReadLine.pm 
#	with ...

ENV DEBIAN_FRONTEND noninteractive

ENV SAMTOOLS_VERSION="1.5"
ENV BOWTIE2_VERSION="2.3.2"
WORKDIR $HOME
 
RUN apt-get update && apt-get install -y apt-utils dialog bzip2 gcc gawk zlib1g-dev libbz2-dev liblzma-dev libcurl4-openssl-dev make libssl-dev libncurses5-dev zip g++ git libtbb-dev wget && apt-get clean

RUN cd / && wget https://github.com/samtools/htslib/releases/download/${SAMTOOLS_VERSION}/htslib-${SAMTOOLS_VERSION}.tar.bz2 && tar xvfj htslib-${SAMTOOLS_VERSION}.tar.bz2 && cd htslib-${SAMTOOLS_VERSION} && ./configure && make && make install && cd ~ && /bin/rm -rf /htslib-${SAMTOOLS_VERSION}*
 
RUN cd / && wget https://github.com/samtools/samtools/releases/download/${SAMTOOLS_VERSION}/samtools-${SAMTOOLS_VERSION}.tar.bz2 && tar xvfj samtools-${SAMTOOLS_VERSION}.tar.bz2 && cd samtools-${SAMTOOLS_VERSION} && ./configure && make && make install && cd ~ && /bin/rm -rf /samtools-${SAMTOOLS_VERSION}*

RUN cd / && wget https://sourceforge.net/projects/bowtie-bio/files/bowtie2/${BOWTIE2_VERSION}/bowtie2-${BOWTIE2_VERSION}-source.zip/download -O bowtie2-${BOWTIE2_VERSION}-source.zip && unzip bowtie2-${BOWTIE2_VERSION}-source.zip && cd bowtie2-${BOWTIE2_VERSION} && make && make install && cd ~ && /bin/rm -rf /bowtie2-${BOWTIE2_VERSION}*

RUN cd ~ && git clone http://github.com/unreno/chimera && cd chimera && ln -s Makefile.example Makefile && make BASE_DIR="/usr/local" install && cd ~ && /bin/rm -rf chimera
```



Environment variables set in the Dockerfile are set in the created docker image.










##	How to run a container that cleans up after itself?

The -rm option will cause the container to remove itself upon completion.


I cannot figure out how to have a docker container shut itself down from within. No “sudo shutdown -h now” or “sudo halt”.


It may be moot. Simply run the command from the command line when starting docker with the --rm option. No “sudo”. No “halt”. No “shutdown”. Just “exit” and poof.





Just run a command and exit leaving no running container …


```
docker run --rm chimera echo hello there
```



##	How to delete a container?

```
docker rm CONTAINER_ID
```


##	How to delete all containers?

```
docker rm $( docker ps -qa )
```


##	How to delete a local image?


```
docker rmi IMAGE_ID_OR_NAME
```

##	How to delete all local images?

```
docker rmi $( docker images -q )
```





##	How to upload to CGC or Seven Bridges?


First off, you’ll need a developer’s token, so login, go to the Developer link and create one.

Then login with your username.

When prompted for a password, paste your developer token string.

```
#	Seven Bridges
#	http://docs.sevenbridges.com/docs/get-your-authentication-token
#	images.sbgenomics.com
#	or CGC
#	https://cgc.sbgenomics.com/developer#token
#	cgc-images.sbgenomics.com
#
#	Token will be like .. b16e7a9ca5a143bea7d0048636aac600
#	login is only required to download or upload, otherwise irrelevant
```

```
docker login --username wendt2017 cgc-images.sbgenomics.com
```

In a local, empty dir, create your Dockerfile and build your docker image with desired tags.

```
docker build -t cgc-images.sbgenomics.com/wendt2017/chimera:20170721.0 -t cgc-images.sbgenomics.com/wendt2017/chimera:latest .
```

Then push this image to your CGC repository.

```
docker push cgc-images.sbgenomics.com/wendt2017/chimera:20170721.0
docker push cgc-images.sbgenomics.com/wendt2017/chimera:latest
```

Feel free to delete your local images. You should still be able to start the image locally with ...

```
docker run -it cgc-images.sbgenomics.com/wendt2017/chimera:20170721.0
```

Apparently, you can’t delete these images and they are publicly usable, but NOT publicly browsable, so be discrete.

Still can’t see image tags on CGC so no idea which one is there unless you know.















This is your new virtual instance. Update it. Upgrade it. Install whatever packages you’ll need.

```
apt update
apt full-upgrade


apt install bedtools git zip wget curl bzip2 make zlib1g-dev libbz2-dev liblzma-dev gcc  g++ libncurses5-dev libtbb-dev
```




Install all your other goodies

samtools (way outdated apt so need source)

```
wget https://sourceforge.net/projects/samtools/files/samtools/1.4.1/samtools-1.4.1.tar.bz2/download -O samtools-1.4.1.tar.bz2
wget https://sourceforge.net/projects/samtools/files/samtools/1.4.1/bcftools-1.4.1.tar.bz2/download -O bcftools-1.4.1.tar.bz2
wget https://sourceforge.net/projects/samtools/files/samtools/1.4.1/htslib-1.4.1.tar.bz2/download -O htslib-1.4.1.tar.bz2
```

Bowtie2
```
wget https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.3.2/bowtie2-2.3.2-source.zip/download -O bowtie2-2.3.2-source.zip
wget https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.3.2/bowtie2-2.3.2-linux-x86_64.zip/download -O bowtie2-2.3.2-linux-x86_64.zip
```

Your scripts...

```
GIT CLONE ...
make install
```




Create a new image from your running container

```
docker commit 56e214e76b9e testimage

docker images
```


Save this image as a tar file for export

```
docker save testimage --output testimage.tar
```


Copy your image elsewhere and import

```
docker load --input testimage.tar 
sha256:8bd605fbf6f92539378e98bc48d596beab39f0e23cceb54001dd61f8b5f37
```




start a new instance with your image

```
docker run -ti testimage
```






Want to delete dependent images.

Need to save desired image.

Then delete desired image and dependent images.

Then load saved desired image.

Dependent images will show as layers during load, but not in images list.







`docker run -ti ubuntu`


```
apt update
apt full-upgrade
apt install dialog bedtools git zip wget curl bzip2 make zlib1g-dev libbz2-dev liblzma-dev gcc  g++ libncurses5-dev libtbb-dev python-pip libcurl4-openssl-dev  apt-utils gawk libssl-dev

#	apt install r-base (takes about 650MB!)
#	apt install mariadb-server mariadb-common mariadb-client libmariadb2 ( 150MB)
#	apt install php	( 50MB )


#	400 MB
pip install --upgrade awscli deeptools grip pip


#	100 MB
cd ~
wget https://sourceforge.net/projects/samtools/files/samtools/1.4.1/htslib-1.4.1.tar.bz2/download -O htslib-1.4.1.tar.bz2
tar xvfj htslib-1.4.1.tar.bz2
cd htslib-1.4.1
./configure
make
make install
cd ~
/bin/rm -rf htslib-1.4.1 htslib-1.4.1.tar.bz2


#	negligible
cd ~
wget https://sourceforge.net/projects/samtools/files/samtools/1.4.1/samtools-1.4.1.tar.bz2/download -O samtools-1.4.1.tar.bz2
tar xvfj samtools-1.4.1.tar.bz2
cd samtools-1.4.1
./configure
make
make install
cd ~
/bin/rm -rf samtools-1.4.1 samtools-1.4.1.tar.bz2


#	negligible
cd ~
wget https://sourceforge.net/projects/samtools/files/samtools/1.4.1/bcftools-1.4.1.tar.bz2/download -O bcftools-1.4.1.tar.bz2
tar xvfj bcftools-1.4.1.tar.bz2
cd bcftools-1.4.1
make
make install
cd ~
/bin/rm -rf bcftools-1.4.1 bcftools-1.4.1.tar.bz2


#	40 MB
cd ~
wget https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.3.2/bowtie2-2.3.2-source.zip/download -O bowtie2-2.3.2-source.zip
unzip bowtie2-2.3.2-source.zip
cd bowtie2-2.3.2
make
make install
cd ~
/bin/rm -rf bowtie2-2.3.2 bowtie2-2.3.2-source.zip
```






##	Copy a file from host to container:

```
docker cp foo.txt 72ca2488b353:/foo.txt
```


##	Copy a file from Docker container to host:

```
docker cp 72ca2488b353:/foo.txt foo.txt
```





