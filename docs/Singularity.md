
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

cumulative build time (17h21m12.795s) larger than allowed (16h40m0s)Build 6218141f4a28720818657ba7 cancelled: cumulative build time (17h21m12.795s) larger than allowed (16h40m0s)






##	Installing Singularity on a Mac

https://sylabs.io/guides/3.0/user-guide/installation.html#install-on-windows-or-mac


```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew install --cask virtualbox && brew install --cask vagrant && brew install --cask vagrant-manager && brew install --cask vagrant-vmware-utility

#	REBOOT

mkdir ~/vm-singularity && cd ~/vm-singularity

vagrant init sylabs/singularity-3.0-ubuntu-bionic64 

vagrant up

vagrant ssh


singularity version

```


OK. That seems to work now.

Now need to learn how to share a folder with the virtual box so that I can build an image from a definition.









