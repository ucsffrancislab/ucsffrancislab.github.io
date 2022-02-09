
#	Singularity


If you have a singularity image, it is pretty straight forward. iMOKA is run solely through an image.

Passing files into and out of the image is usually done via a bound path.
This can be done with a parameter or an environment variable.
For some reason, I remember having an issue using iMOKA and it needed the environment variable.
Perhaps the code explicitly uses the environment variable.

```
export SINGULARITY_BINDPATH=/francislab,/scratch

singularity exec ${img} iMOKA_core reduce \
	--input ${TMPDIR}/matrix.json \
	--output ${TMPDIR}/reduced.matrix
```

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

singularity build --remote  TEfinder.simg TEfinder

singularity exec TEfinder.simg ls /
bin  boot  c4  data  dev  environment  etc  home  lib  lib64  media  mnt  opt  proc  root  run	sbin  singularity  srv	sys  tmp  usr  var

singularity exec TEfinder.simg whoami
gwendt
```

Worked.

