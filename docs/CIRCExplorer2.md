
#	CIRCExplorer2


This seems to require a very particular set of dependencies in a particular order.
My final set for TIPCC includes the following.
I'm not entirely confident that this is accurate as some errors did not repeat and some work was being done on the cluster at the time.
Nevertheless, ...




For some reason, without the -f option, the second command fails. The first doesn't care. Buffer size?
Don't know exactly why, but "CIRCexplorer2 align" may actually work now. Nope just failed later!!!!

```
bowtie_header_cmd += ["-f"] 

/home/gwendt/.local/bowtie-1.2.3/bowtie --sam /francislab/data1/refs/CIRCexplorer2/bowtie1_index -f /dev/null

/home/gwendt/.local/bowtie-1.2.3/bowtie --sam /francislab/data1/raw/20201127-EV_CATS/CIRCexplorer2/L6_R1/tophat_fusion/tmp/segment_juncs -f /dev/null
```




```
pip install --upgrade --user pysam==0.15.2
export PATH="$HOME/.local/tophat-2.1.0/:$PATH"
module load samtools/1.7
module load bowtie2/2.3.4.1
module load bowtie/1.2.2
module load python/2.7.10
```

Works. No idea why it failed before. So confused.

