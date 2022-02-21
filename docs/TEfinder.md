
#	TEfinder

[TEfinder](https://github.com/VistaSohrab/TEfinder)


Given the limitations and difficulties, I've been creating Singularity containers for these new pipelines.


I've created a container and image (remotely) with ...

```
cd ~/github/ucsffrancislab/genomics/singularity

singularity remote login --tokenfile ~/sylabs-token 

singularity build --remote TEfinder.img TEfinder
```


I ran the following on a Friday.
```
cp /francislab/data1/refs/sources/igv.org.genomes/hg38/rmsk/hg38_rmsk_LTR.gtf

awk -F '\t' '{print $9}' hg38_rmsk_LTR.gtf  | awk -F '"' '{print $2}' | sort | uniq > hg38_rmsk_LTR.txt

export SINGULARITY_BINDPATH=/francislab

singularity exec TEfinder.img TEfinder --threads 8 -alignment /francislab/data1/raw/20200909-TARGET-ALL-P2-RNA_bam/bam/10-PAUCDY-09A-01R.bam -fa ${PWD}/hg38.fa -gtf ${PWD}/hg38_rmsk_LTR.gtf -te ${PWD}/hg38_rmsk_LTR.txt
```
I had to kill it on Monday.

It ran all weekend with many, many processes like ...
```
/bin/bash /usr/bin/TEfinder -alignment /francislab/data1/raw/20200909-TARGET-ALL-P2-RNA_bam/bam/10-PAUCDY-09A-01R.

bedtools intersect -abam TEfinder_20220218073610/Alignments.bam -b TEfinder_20220218073610/LTR17/LTR17_TE.gff -wa
```
I think that there is a lot of cross comparison. Limiting the elements of interest list could speed this up.


Also, need to find a way to skip sorting. Waste of time to sort a already-sorted bam. Can take hours.




