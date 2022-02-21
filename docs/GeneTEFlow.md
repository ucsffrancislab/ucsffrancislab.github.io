

#	GeneTEFlow

[GeneTEFlow](https://github.com/zhongw2/GeneTEFlow)


Given the limitations and difficulties, I've been creating Singularity containers for these new pipelines.



This one comes with several Dockerfiles. I'm trying to convert them to Singularity containers, but having some trouble getting them built.


Testing spython to convert a Dockerfile into a Singularity definition file.
```
python3 -m pip install --user --upgrade spython
```

I'd also like to dump any conda usage.

```
spython recipe Dockerfile >> Singularity
```

Starting with Analysis.
That seemed to work. Threw a couple errors due to extra space in command `COPY ngsdb  /RNASeq/ngsdb`.
Unfortunately, it copies some local files which doesn't work when building remotely.

Will need to change the copy from local to the online github repo.

Even the existing conda environment won't install.




