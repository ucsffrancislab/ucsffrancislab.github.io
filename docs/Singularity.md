
#	Singularity


If you have a singularity image, it is pretty straight forward. iMOKA is run solely through an image.

Passing files into and out of the image is usually done via a bound path.
This can be done with a parameter or an environment variable.
iMOKA needed the environment variable.
Perhaps the code explicitly uses the environment variable.

```
export SINGULARITY_BINDPATH=/francislab,/scratch

singularity exec ${img} iMOKA_core reduce \
	--input ${TMPDIR}/matrix.json \
	--output ${TMPDIR}/reduced.matrix
```

iMOKA uses Dockerfiles, but singularity to build the container. Not sure why.

Singularity does not natively run on Mac so building on my Mac is not a viable solution.

Building on C4 is tricky. Normally, you need root / sudo permission to build.

There is a `--fakeroot` option, but that doesn't seem to work either.

Then there's the `--remote` option which apparently does.

There are a few issues expected. Let's see.

The build apparently happens remotely, so any files that need to be included in the image need to be in the cloud.


Apparently you need a token from the remote server which will require a user.

I'm using the default server https://cloud.sylabs.io/

I created a user with my Google login.

Clicked on my username, then Access Tokens, Create New Access Token, Download Token.


```
singularity remote login --tokenfile ~/sylabs-token 

singularity build --remote TEfinder.img TEfinder
```

This worked.


Building on a remote server leaves builds and running instances on said server.
There seems to be a limit after which build will silently hang.
You'll need to login to the remote build server and cleanup.

C4 is developing a singularity build server.

```
singularity exec TEfinder.img ls /
bin  boot  c4  data  dev  environment  etc  home  lib  lib64  media  mnt  opt  proc  root  run	sbin  singularity  srv	sys  tmp  usr  var

singularity exec TEfinder.img whoami
gwendt
```


The C4 environment makes bash scripts to error but still run?
```
singularity exec TEfinder.img TEfinder

/bin/bash: BASH_FUNC_ml(): line 0: syntax error near unexpected token `)'
/bin/bash: BASH_FUNC_ml(): line 0: `BASH_FUNC_ml() () {  eval $($LMOD_DIR/ml_cmd "$@")'
/bin/bash: error importing function definition for `BASH_FUNC_ml'
/bin/bash: BASH_FUNC_module(): line 0: syntax error near unexpected token `)'
/bin/bash: BASH_FUNC_module(): line 0: `BASH_FUNC_module() () {  eval $($LMOD_CMD bash "$@") && eval $(${LMOD_SETTARG_CMD:-:} -s sh)'
/bin/bash: error importing function definition for `BASH_FUNC_module'
One or more required parameters are missing.
example: TEfinder -alignment sample.bam -fa reference.fa -gtf TEs.gtf -te List_of_TEs.txt
```

There are a number of environment variables that could be the issue.
```
singularity exec TEfinder.img env
```

Call with --cleanenv until I figure out exactly what the issue is.
```
singularity exec --cleanenv TEfinder.img TEfinder

singularity exec --cleanenv TEfinder.img env
```


Not sure why iMOKA's preprocess.sh doesn't complain.
Perhaps cause its a Dockerfile build?
Or simply a different linux core?
bash is 5.0.17

Using 
```
Bootstrap: docker
From: ubuntu:bionic
```
instead of 
```
Bootstrap: shub
From: singularityhub/ubuntu
```
increase the bash version from 4.3.8 to 4.4.20
and the bash function errors go away. 
Not sure if there are other differences.

Even better
```
Bootstrap: library
From: ubuntu:20.04
```
bash 5.0

No errors or --cleanenv needed.
Looks cleaner, more generic and more current.


Building on this remote server, at least using the free version, there appears to be a size limit.
The account has a limit of 10GB. Apparently there can be a lot of remnants left that take up space.
There was 7GB of stuff hidden somewhere in my account.

Also, a limit on build time. Not sure what the denominator is. Per week? Month?

```
cumulative build time (17h21m12.795s) larger than allowed (16h40m0s)Build 6218141f4a28720818657ba7 cancelled: cumulative build time (17h21m12.795s) larger than allowed (16h40m0s)
```






##	Installing Singularity on a Mac

https://sylabs.io/guides/3.0/user-guide/installation.html#install-on-windows-or-mac


```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew install --cask virtualbox && brew install --cask vagrant && brew install --cask vagrant-manager && brew install --cask vagrant-vmware-utility
```

REBOOT your main machine.

The current dir is shared with the virtual machine as /vagrant.

```
cd ~/github/ucsffrancislab/genomics/singularity

#vagrant init sylabs/singularity-3.0-ubuntu-bionic64 
#vagrant init sylabs/singularity-3.7-ubuntu-bionic64 
vagrant init sylabs/singularity-ce-3.9-ubuntu-bionic64 

vagrant up

vagrant ssh

singularity version

cd /vagrant/

sudo singularity build McClintock.img McClintock | tee McClintock.out
sudo singularity build RepEnrich2.img RepEnrich2 | tee RepEnrich2.out
sudo singularity build SQuIRE.img SQuIRE | tee SQuIRE.out
sudo singularity build TEfinder.img TEfinder | tee TEfinder.out
sudo singularity build TEtools.img TEtools | tee TEtools.out
sudo singularity build TEtranscripts.img TEtranscripts | tee TEtranscripts.out
sudo singularity build xTea.img xTea | tee xTea.out

vagrant destroy
```


Old or failed
```
sudo singularity build GeneTEFlow.img GeneTEFlow | tee GeneTEFlow.out
```


So the current working dir is shared with vagrant, so 


```
https://vagrantcloud.com/search

2.24	Ubuntu 16.04 LTS (Xenial Xerus)
2.25	Ubuntu 16.10 (Yakkety Yak)
2.26	Ubuntu 17.04 (Zesty Zapus)
2.27	Ubuntu 17.10 (Artful Aardvark)
2.28	Ubuntu 18.04 LTS (Bionic Beaver)  <------ Nothing newer?
2.29	Ubuntu 18.10 (Cosmic Cuttlefish)
2.30	Ubuntu 19.04 (Disco Dingo)
2.31	Ubuntu 19.10 (Eoan Ermine)
2.32	Ubuntu 20.04 LTS (Focal Fossa)
2.33	Ubuntu 20.10 (Groovy Gorilla)
2.34	Ubuntu 21.04 (Hirsute Hippo)
2.35	Ubuntu 21.10 (Impish Indri)
2.36	Ubuntu 22.04 LTS (Jammy Jellyfish)
```




##	Singularity on AWS Instance


https://raw.githubusercontent.com/ucsffrancislab/genomics/master/aws/ec2_install_singularity.bash


https://github.com/apptainer/singularity/blob/master/INSTALL.md

```
sudo apt-get update

sudo apt-get install -y \
    build-essential \
    libseccomp-dev \
    pkg-config \
    squashfs-tools \
    cryptsetup \
    curl wget git 

sudo apt-get -y autoremove

export GOVERSION=1.17.3 OS=linux ARCH=amd64  # change this as you need

wget -O /tmp/go${GOVERSION}.${OS}-${ARCH}.tar.gz \
  https://dl.google.com/go/go${GOVERSION}.${OS}-${ARCH}.tar.gz
sudo tar -C /usr/local -xzf /tmp/go${GOVERSION}.${OS}-${ARCH}.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

git clone https://github.com/hpcng/singularity.git
cd singularity

git checkout v3.8.4

./mconfig
cd ./builddir
make
sudo make install

sudo chmod o+r /usr/local/etc/singularity/capability.json

singularity --version
```
